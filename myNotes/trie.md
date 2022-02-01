# Merkle Patricia Tree

## Merkle树

<https://www.cnblogs.com/fengzhiwu/p/5524324.html>

![](https://images2015.cnblogs.com/blog/834896/201605/834896-20160527163537178-321412097.png)

Merkle Tree，通常也被称作Hash Tree，顾名思义，就是存储hash值的一棵树。Merkle树的叶子是数据块(例如，文件或者文件的集合)的hash值。非叶节点是其对应子节点串联字符串的hash。Merkle Tree的主要作用是当我拿到Top Hash的时候，这个hash值代表了整颗树的信息摘要，当树里面任何一个数据发生了变动，都会导致Top Hash的值发生变化。 而Top Hash的值是会存储到区块链的区块头里面去的， 区块头是必须经过工作量证明。 这也就是说我只要拿到一个区块头，就可以对区块信息进行验证。

Merkle Tree的特点

1.  MT是一种树，大多数是二叉树，也可以多叉树，无论是几叉树，它都具有树结构的所有特点；

2.  Merkle Tree的叶子节点的value是数据集合的单元数据或者单元数据HASH。

3.  非叶子节点的value是根据它下面所有的叶子节点值，然后按照Hash算法计算而得出的。

## Trie树

 Trie，又经常叫前缀树，字典树，也是一种key-value的存储结构。比如有下面这些单词需要排成trie的数据结构，gennral、genesis、god、go、good。

![trie01.drawio.png](https://github.com/hrt014pocky/pocky/blob/main/pictures/trie01.drawio.png?raw=true "trie01.drawio.png")

Trie树的特点可以归纳为：

1.  trie的每个节点的分支数目取决于key中的元素的取值范围

2.  trie的查找效率取决于key的长度

3.  trie不会存在哈希碰撞

4.  输入不变，最终得到的trie树是唯一的

5.  更新操作是局部性的

## Patricia Tries

Patricia Tries是一种路径压缩的trie。直观上看树的高度明显变小了，访问内存的次数就少了，效率提高。以太坊中的账户地址分布是非常稀疏的。

![PatriciaTries.drawio.png](https://github.com/hrt014pocky/pocky/blob/main/pictures/PatriciaTries.drawio.png?raw=true "PatriciaTries.drawio.png")

### 以太坊的MPT

![worldstatetrie.png](https://github.com/hrt014pocky/pocky/blob/main/pictures/worldstatetrie.png?raw=true "worldstatetrie.png")

## 课程视频笔记

### 以太坊状态数据结构推测

以太坊是基于账户的账本，需要完成从账户地址到账户状态的映射，address=>state。address是一个160bits的地址，state包括余额、nonce、code等。首先想到是使用哈希表。

#### 哈希表

首先想到是使用哈希表，更新、插入、查询都在这个哈希表中，一个address的key对应一个state的value。每个全节点在本地维护一个哈希表，打包交易时构建一颗merkle tree，根哈希放在区块头。

如果需要提供一个merkle proof，怎么提供？发起交易时，要查询账户余额正确性怎么查询？一种方法是，使用这个哈希表构建一颗merkle tree，把根哈希存放在区块头中，公布出去，只要这个根哈希正确，那么数据就不会被篡改。这样有什么问题呢？新区块发布，当有一笔新的交易时，执行后必然会改变哈希表的内容，又需要重新构建一颗账户地址到账户状态的merkle tree。但是以太坊的账户数量级是巨大的，再构建一颗merkle tree代价太大了。除了证明账户余额外，merkle proof还有另外一个很重要的作用就是维护各个全节点之间状态的一致性。

那么我们考虑第二种方法，不用哈希表，直接构建一颗merkle tree

#### merkle tree

直接构建一颗merkle tree，把所有的账户放在merkle tree中，需要修改的时候直接该merkle tree。需要修改的只是很小一部分的账户信息，只需要修改很小一部分的merkle tree。这个结构的问题在于merkle tree没有提供一个快速高效的查找和更新的方法。还有一个问题就是，这个merkle tree需不需要排序？假如不排序，这些账户组成一个merkle tree的叶子节点是账户信息，不规定叶子节点在merkle tree中出现的顺序，在每个全节点的构建的merkle tree是不唯一的，计算出来的根哈希也是不唯一的，也就无法达成共识。所以每个10几秒，发布新区块都会构建一颗不一样的merkle tree。而大多数的账户状态是不变的，账户的数量级很大，所以每次发布新区块都构建新merkle tree是不现实的。
那么，排序的merkle tree行不行呢

#### sorted merkle tree

如果有新增的账户怎么办？账户地址是随机的，很可能在merkle tree的中间插入一个叶子节点，后面的tree又需要重构merkle tree。于是又回到了上面同样的问题。sorted merkle tree插入一个账户的代价太大。
