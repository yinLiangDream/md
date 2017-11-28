今天要说的是最简单的 for 循环，一个简单的 for 循环看似没有任何优化的意义，但实质上优化前后差距挺大的，那么该如何优化呢？

从最简单的遍历数组说起。


```javascript
// 定义一个数组arr（假设是从后台返回的数据）
let i = 0;
let arr = [];
while (i < 50) {
    arr.push(i);
    i++;
}
```
如果我们想从数组 arr 中取出数据，就必须要进行遍历，普遍的做法是：

```javascript
for (let i = 0; i < arr.length; i++) {
    // arr[i]
}
```
但其实这样的写法遍历是最慢的，他要经过两次迭代，第一次是 i 的迭代，每次都要判断 i 是否小于 arr.length，第二次是 arr 的迭代，每次循环 arr 都会调用底层的迭代器，对长度进行计算，这样循环的效率非常低，时间空间复杂度为 O[n^2]。

下面进行优化，看看两者到底有什么区别：

```javascript
for (let i = 0, len = arr.length; i < len; i++) {
    // arr[i]
}
```
区别就是，整个循环当中，我们预存了 len 来保存数组的长度，这样不需要每次循环都调用底层迭代器，调用一次即可，这样的时间空间复杂度为 O[n+1]。

但是这并不是最完美的，因为会多了一次迭代操作，那么该如何进行优化呢？

以下进行进一步优化：

```javascript
for (let i = 0, item; item = arr[i++];) {
    // item
}
```
这次迭代的时间空间复杂度为 O[n] ，完美做到了每次一迭代没有通过长度进行判断，而是直接通过下标进行取值的方式映射到了循环体内部。

最后用5万条数据进行测试三种方式的循环时间：


```javascript
// 定义一个数组arr（假设是从后台返回的数据）
let index = 0;
let arr = [];
while (index < 50000) {
    arr.push(index);
    index++;
}

console.time('one');
for (let i = 0; i < arr.length; i++) {
    // arr[i]
}
console.timeEnd('one');
// one: 2.09765625ms

console.time('two');
for (let i = 0, len = arr.length; i < len; i++) {
    // arr[i]
}
console.timeEnd('two');
// two: 0.839111328125ms

console.time('three');
for (let i = 0, item; item = arr[i++];) {
    // arr[i]
}
console.timeEnd('three');
// three: 0.004150390625ms
```
可以看出，传统的写法需要 2ms 才能进行迭代完成，优化后只需 0.8ms ，最终优化的结果只需要 0.004ms 就能迭代完成，性能大大提升，在数据量大的情况下，效果显而易见。

另外一种优化策略是“达夫设备”，但是随着市面上浏览器性能的逐渐增强，老版本的浏览器运用达夫设备优化性能能得到大幅度的提升，而新版的浏览器引擎肯定对循环迭代语句进行了更强的优化，所以达夫设备能实现的优化效果日趋减弱甚至于没有。而在查阅资料的过程中，有人提出 while 循环的效果和达夫设备不相上下，所以这里就不做介绍了。