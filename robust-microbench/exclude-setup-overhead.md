### Exclude Setup Overhead
Some benchmarks need to perform additional setup work before their main workload. Historically, this was dealt with by sizing the main workload so that it dwarfs the setup, making it negligible. Setup is performed before the main work loop, and its impact is further lessened by amortizing it over the `N` measured iterations.

Since we no longer measure with `N>1`, the effect of setup becomes more pronounced. One of the most extreme cases, benchmark [`ReversedArray`](https://github.com/apple/swift/blob/master/benchmark/single-source/ReversedCollections.swift) clearly demonstrates the amortization of the setup overhead as `num-iters` increase.<sup>[7](chart.html?b=ReversedArray&v=iters&ry=188.6+376.6&rx=0+1087235)</sup>

<iframe src="chart.html?b=ReversedArray&v=iters&hide=navigation+zoom+outliers+plots+stats+overhead+note&ry=188.6+376.6&rx=0+1087235" name="ReversedArray+iters+raw" frameborder="0" width="100%" height="430"></iframe>

The setup overhead is a systematic measurement error, that can be detected and corrected for, when measuring with different `num-iters`. Given two measurements performed with `i` and `j` iterations, that reported corresponding runtimes `ti` and `tj`, the setup overhead can be computed as follows:

```setup = (i * j * (ti - tj)) / (j - i)```

We can detect the setup overhead by picking smallest minimum from series with same `num-iters` and using the above formula where `i=1, j=2`. In the *a10R* series from `ReversedArray` it gives us 134µs of setup overhead (or 41.4% of the minimal value).<sup>[7](chart.html?b=ReversedArray&v=iters&ry=188.6+376.6)</sup>

<iframe src="chart.html?b=ReversedArray&v=a10R&hide=navigation+zoom+outliers+plots+stats+note&ry=188.6+376.6" name="ReversedArray+a10R+raw" frameborder="0" width="100%" height="430"></iframe>

We can normalize the series with different `num-iters` by subtracting the corresponding fraction of the setup from each sample. The median value after we exclude the setup overhead is 190µs which exactly matches the baseline from the [i0 Series](chart.html?b=ReversedArray&v=i0).<sup>[8](chart.html?b=ReversedArray&v=a10R&ry=188.6+376.6&overhead=true)</sup>

<iframe src="chart.html?b=ReversedArray&v=a10R&hide=navigation+zoom+outliers+plots+stats+note&ry=188.6+376.6&overhead=true" name="ReversedArray+a10R+corrected" frameborder="0" width="100%" height="430"></iframe>

However, our ability to detect overhead with this technique depends on its size relative to the sample variance. Some small overhead is apparent only in the *a* Series and gets lost in noisier series. Another issue is that for larger overheads, when we subtract it, the corrected sample has higher variance relative to other benchmarks with similar runtimes. This is because the sample dispersion is always relative to the runtime. When we subtract the constant overhead we get better runtime, but same dispersion (i.e. the IQR and standard deviation are unchanged).

Benchmarks from Swift Benchmark Suite with setup overhead in % relative to the runtime.
*The % links open the `chart.html`; Hover over the links for absolute values in µs*:

| Benchmark | a10 | a10R | a12 |
|---|---|---|---|
| ClassArrayGetter | [100%](chart.html?b=ClassArrayGetter&v=a10 "5974µs") | [100%](chart.html?b=ClassArrayGetter&v=a10R "6000µs") | [99%](chart.html?b=ClassArrayGetter&v=a12 "5950µs") |
| ReversedDictionary | [99%](chart.html?b=ReversedDictionary&v=a10 "48700µs") | [99%](chart.html?b=ReversedDictionary&v=a10R "48750µs") | [100%](chart.html?b=ReversedDictionary&v=a12 "48896µs") |
| ArrayOfGenericRef | [49%](chart.html?b=ArrayOfGenericRef&v=a10 "16292µs") | [50%](chart.html?b=ArrayOfGenericRef&v=a10R "16538µs") | [50%](chart.html?b=ArrayOfGenericRef&v=a12 "16664µs") |
| ArrayOfPOD | [50%](chart.html?b=ArrayOfPOD&v=a10 "622µs") | [50%](chart.html?b=ArrayOfPOD&v=a10R "622µs") | [50%](chart.html?b=ArrayOfPOD&v=a12 "622µs") |
| ArrayOfGenericPOD | [50%](chart.html?b=ArrayOfGenericPOD&v=a10 "662µs") | [50%](chart.html?b=ArrayOfGenericPOD&v=a10R "664µs") | [50%](chart.html?b=ArrayOfGenericPOD&v=a12 "664µs") |
| Chars | [50%](chart.html?b=Chars&v=a10 "3194µs") | [50%](chart.html?b=Chars&v=a10R "3190µs") | [50%](chart.html?b=Chars&v=a12 "3194µs") |
| ArrayOfRef | [49%](chart.html?b=ArrayOfRef&v=a10 "16018µs") | [50%](chart.html?b=ArrayOfRef&v=a10R "16126µs") | [49%](chart.html?b=ArrayOfRef&v=a12 "16000µs") |
| ReversedArray | [41%](chart.html?b=ReversedArray&v=a10 "134µs") | [41%](chart.html?b=ReversedArray&v=a10R "134µs") | [41%](chart.html?b=ReversedArray&v=a12 "134µs") |
| DictionaryGroupOfObjects | [36%](chart.html?b=DictionaryGroupOfObjects&v=a10 "5872µs") | [34%](chart.html?b=DictionaryGroupOfObjects&v=a10R "5614µs") | [36%](chart.html?b=DictionaryGroupOfObjects&v=a12 "5934µs") |
| ArrayAppendStrings | [24%](chart.html?b=ArrayAppendStrings&v=a10 "23408µs") | [26%](chart.html?b=ArrayAppendStrings&v=a10R "24826µs") | [25%](chart.html?b=ArrayAppendStrings&v=a12 "24228µs") |
| SetIsSubsetOf_OfObjects | [20%](chart.html?b=SetIsSubsetOf_OfObjects&v=a10 "462µs") | [20%](chart.html?b=SetIsSubsetOf_OfObjects&v=a10R "460µs") | [20%](chart.html?b=SetIsSubsetOf_OfObjects&v=a12 "460µs") |
| SortSortedStrings | [15%](chart.html?b=SortSortedStrings&v=a10 "436µs") | [15%](chart.html?b=SortSortedStrings&v=a10R "440µs") | [15%](chart.html?b=SortSortedStrings&v=a12 "440µs") |
| IterateData | [10%](chart.html?b=IterateData&v=a10 "518µs") | [13%](chart.html?b=IterateData&v=a10R "690µs") | [11%](chart.html?b=IterateData&v=a12 "576µs") |
| SuffixArray | [10%](chart.html?b=SuffixArray&v=a10 "6µs") | [7%](chart.html?b=SuffixArray&v=a10R "4µs") | [10%](chart.html?b=SuffixArray&v=a12 "6µs") |
| PolymorphicCalls | [10%](chart.html?b=PolymorphicCalls&v=a10 "6µs") | [10%](chart.html?b=PolymorphicCalls&v=a10R "6µs") | [10%](chart.html?b=PolymorphicCalls&v=a12 "6µs") |
| Phonebook | [9%](chart.html?b=Phonebook&v=a10 "1498µs") | [9%](chart.html?b=Phonebook&v=a10R "1588µs") | [9%](chart.html?b=Phonebook&v=a12 "1556µs") |
| SetIntersect_OfObjects | [8%](chart.html?b=SetIntersect_OfObjects&v=a10 "860µs") | [8%](chart.html?b=SetIntersect_OfObjects&v=a10R "890µs") | [9%](chart.html?b=SetIntersect_OfObjects&v=a12 "950µs") |
| SetIsSubsetOf | [9%](chart.html?b=SetIsSubsetOf&v=a10 "136µs") | [9%](chart.html?b=SetIsSubsetOf&v=a10R "134µs") | [9%](chart.html?b=SetIsSubsetOf&v=a12 "136µs") |
| DropLastArray | [9%](chart.html?b=DropLastArray&v=a10 "4µs") | [9%](chart.html?b=DropLastArray&v=a10R "4µs") | [9%](chart.html?b=DropLastArray&v=a12 "4µs") |
| Dictionary | [7%](chart.html?b=Dictionary&v=a10 "204µs") | [8%](chart.html?b=Dictionary&v=a10R "220µs") | [8%](chart.html?b=Dictionary&v=a12 "234µs") |
| SetIntersect | [8%](chart.html?b=SetIntersect&v=a10 "272µs") | [8%](chart.html?b=SetIntersect&v=a10R "268µs") | [8%](chart.html?b=SetIntersect&v=a12 "272µs") |
| DictionaryOfObjects | [8%](chart.html?b=DictionaryOfObjects&v=a10 "832µs") | [7%](chart.html?b=DictionaryOfObjects&v=a10R "780µs") | [7%](chart.html?b=DictionaryOfObjects&v=a12 "746µs") |
| MapReduceShort | [7%](chart.html?b=MapReduceShort&v=a10 "812µs") | [6%](chart.html?b=MapReduceShort&v=a10R "642µs") | [5%](chart.html?b=MapReduceShort&v=a12 "528µs") |
| DropLastArrayLazy | [7%](chart.html?b=DropLastArrayLazy&v=a10 "4µs") | [7%](chart.html?b=DropLastArrayLazy&v=a10R "4µs") | [7%](chart.html?b=DropLastArrayLazy&v=a12 "4µs") |
| StaticArray | [7%](chart.html?b=StaticArray&v=a10 "2µs") | [7%](chart.html?b=StaticArray&v=a10R "2µs") | [7%](chart.html?b=StaticArray&v=a12 "2µs") |
| SuffixArrayLazy | [6%](chart.html?b=SuffixArrayLazy&v=a10 "4µs") | [6%](chart.html?b=SuffixArrayLazy&v=a10R "4µs") | [6%](chart.html?b=SuffixArrayLazy&v=a12 "4µs") |
| MapReduceClass | [6%](chart.html?b=MapReduceClass&v=a10 "716µs") | [4%](chart.html?b=MapReduceClass&v=a10R "498µs") | [4%](chart.html?b=MapReduceClass&v=a12 "482µs") |
| ArrayInClass | [4%](chart.html?b=ArrayInClass&v=a10 "134µs") | [5%](chart.html?b=ArrayInClass&v=a10R "154µs") | [5%](chart.html?b=ArrayInClass&v=a12 "144µs") |
| SubstringFromLongString | [4%](chart.html?b=SubstringFromLongString&v=a10 "2µs") | [4%](chart.html?b=SubstringFromLongString&v=a10R "2µs") | [4%](chart.html?b=SubstringFromLongString&v=a12 "2µs") |
| DropFirstArray | [4%](chart.html?b=DropFirstArray&v=a10 "6µs") | [4%](chart.html?b=DropFirstArray&v=a10R "6µs") | [4%](chart.html?b=DropFirstArray&v=a12 "6µs") |
| PrefixArray | [3%](chart.html?b=PrefixArray&v=a10 "4µs") | [3%](chart.html?b=PrefixArray&v=a10R "4µs") | [3%](chart.html?b=PrefixArray&v=a12 "4µs") |
| SuffixAnyCollection | [3%](chart.html?b=SuffixAnyCollection&v=a10 "2µs") | [3%](chart.html?b=SuffixAnyCollection&v=a10R "2µs") | [3%](chart.html?b=SuffixAnyCollection&v=a12 "2µs") |
| DropLastAnyCollection | [3%](chart.html?b=DropLastAnyCollection&v=a10 "2µs") | [3%](chart.html?b=DropLastAnyCollection&v=a10R "2µs") | [3%](chart.html?b=DropLastAnyCollection&v=a12 "2µs") |
| SetUnion_OfObjects | [3%](chart.html?b=SetUnion_OfObjects&v=a10 "1002µs") | [2%](chart.html?b=SetUnion_OfObjects&v=a10R "652µs") | [3%](chart.html?b=SetUnion_OfObjects&v=a12 "1032µs") |
| PrefixArrayLazy | [3%](chart.html?b=PrefixArrayLazy&v=a10 "4µs") | [3%](chart.html?b=PrefixArrayLazy&v=a10R "4µs") | [3%](chart.html?b=PrefixArrayLazy&v=a12 "4µs") |
| UTF8Decode | [2%](chart.html?b=UTF8Decode&v=a10 "36µs") | [2%](chart.html?b=UTF8Decode&v=a10R "48µs") | [2%](chart.html?b=UTF8Decode&v=a12 "40µs") |
| PrefixWhileArrayLazy | [2%](chart.html?b=PrefixWhileArrayLazy&v=a10 "4µs") | [2%](chart.html?b=PrefixWhileArrayLazy&v=a10R "4µs") | [2%](chart.html?b=PrefixWhileArrayLazy&v=a12 "4µs") |
| DropFirstArrayLazy | [2%](chart.html?b=DropFirstArrayLazy&v=a10 "4µs") | [2%](chart.html?b=DropFirstArrayLazy&v=a10R "4µs") | [2%](chart.html?b=DropFirstArrayLazy&v=a12 "4µs") |
| SetExclusiveOr_OfObjects | [2%](chart.html?b=SetExclusiveOr_OfObjects&v=a10 "742µs") | [2%](chart.html?b=SetExclusiveOr_OfObjects&v=a10R "882µs") | [2%](chart.html?b=SetExclusiveOr_OfObjects&v=a12 "854µs") |
| DropFirstSequenceLazy | [2%](chart.html?b=DropFirstSequenceLazy&v=a10 "180µs") | [2%](chart.html?b=DropFirstSequenceLazy&v=a10R "178µs") | [2%](chart.html?b=DropFirstSequenceLazy&v=a12 "190µs") |
| DropWhileArrayLazy | [1%](chart.html?b=DropWhileArrayLazy&v=a10 "6µs") | [1%](chart.html?b=DropWhileArrayLazy&v=a10R "6µs") | [1%](chart.html?b=DropWhileArrayLazy&v=a12 "6µs") |
| DropFirstAnyCollection | [1%](chart.html?b=DropFirstAnyCollection&v=a10 "2µs") | [1%](chart.html?b=DropFirstAnyCollection&v=a10R "2µs") | [1%](chart.html?b=DropFirstAnyCollection&v=a12 "2µs") |
| PrefixAnyCollection | [1%](chart.html?b=PrefixAnyCollection&v=a10 "2µs") | [1%](chart.html?b=PrefixAnyCollection&v=a10R "2µs") | [1%](chart.html?b=PrefixAnyCollection&v=a12 "2µs") |
| MapReduceLazySequence | [1%](chart.html?b=MapReduceLazySequence&v=a10 "2µs") | [1%](chart.html?b=MapReduceLazySequence&v=a10R "2µs") | [1%](chart.html?b=MapReduceLazySequence&v=a12 "2µs") |
| ArrayAppendToFromGeneric | [1%](chart.html?b=ArrayAppendToFromGeneric&v=a10 "16µs") | [1%](chart.html?b=ArrayAppendToFromGeneric&v=a10R "22µs") | [0%](chart.html?b=ArrayAppendToFromGeneric&v=a12 "10µs") |
| PrefixWhileAnyCollectionLazy | [1%](chart.html?b=PrefixWhileAnyCollectionLazy&v=a10 "2µs") | [1%](chart.html?b=PrefixWhileAnyCollectionLazy&v=a10R "2µs") | [1%](chart.html?b=PrefixWhileAnyCollectionLazy&v=a12 "2µs") |

The first two, [`ClassArrayGetter`](https://github.com/apple/swift/blob/master/benchmark/single-source/ClassArrayGetter.swift) and [ReversedDictionary](https://github.com/apple/swift/blob/master/benchmark/single-source/ReversedCollections.swift), are clearly cases of incorrectly written benchmarks where the compiler’s optimizations eliminated the main workload and we are only measuring the setup overhead. Many other are cases of testing a certain feature of `Array` or `Array`-backed type, where the initial creation of the array ends up significant compared to the other fast methods the Array provides.

[PR 12404](https://github.com/apple/swift/pull/12404/commits) has added the ability to perform setup and tear down outside of the measured performance test that is so far used by one benchmark. Rather than automatically correct for the setup overhead, I believe it is best to manually audit the benchmarks from the above table and reassess what should be measured and what should be moved to the setup function outside the main workload.

Previous: [Exclude Outliers](exclude-outliers.md)<br/>
Next: [Detecting Changes](detecting-changes.md)