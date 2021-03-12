---
title: BM算法和KMP算法的Ruby实现
keywords: 'BM,KMP,Ruby'
tags:
  - Ruby
  - 算法
abbrlink: c37b1e7c
date: 2021-03-11 14:34:13
categories:
	- - 数据结构与算法
---

# BM算法和KMP算法的Ruby实现

具体实现的代码放在了我的github上 [Ruby_Algorithm/String_Match at master · Varsion/Ruby_Algorithm (github.com)](https://github.com/Varsion/Ruby_Algorithm/tree/master/String_Match)

## BM算法

全称为 `Boyer-Moore` 算法，其核心思想是利用模式串本身的特点，在模式串中某个字符与主串不能匹配时，将模式串多向后滑动几位，一次来减少不必要的字符比较时间，提高匹配效率。

### 核心原理

BM算法包含两部分：坏字符规则(bad character rule) 和 好后缀规则(good suffix shift)

#### 坏字符规则

从模式串末尾从后往前匹配，当发现某个字符没办法匹配的时候，就将模式串移动到该字符的后一位开始匹配，这个字符就被称为坏字符。

![image-20210311160259549](https://img.varsion.cn/blog-img/20210311160259.png)

该匹配过程中 `C` 就是坏字符串，再次匹配时就可以将字符串直接移动到坏字符的后一位

![image-20210311160324678](https://img.varsion.cn/blog-img/20210311160324.png)

这个时候，模式串最后的一个字符 `D` 还是和主串没办法匹配，但是这个时候就不可以将模式串再向后滑动三位了，因为这个时候 坏字符 `B` 在模式串中是存在的，这种情况下 需要将模式串向后滑动两位 让两个 `B` 上下对齐，再次进行匹配。

![image-20210311160805204](https://img.varsion.cn/blog-img/20210311160805.png)

匹配滑动是有规律的，当发生不匹配的时候，将坏字符对应的模式串中的字符下标记做 `Si` 。如果坏字符在模式串中存在，把这个坏字符在模式串中的下标标记做 `Xi`。如果不存在 就将 `Xi` 记做 -1。模式串向后移动的位数就等于 `Si - Xi`。

**这里说的下标是模式串的下标**。

使用坏字符规则在最好的情况下时间复杂度是非常低的，是 `O(n/m)`。

但是，单纯的使用坏字符串规则是不够的。因为根据 `Si - Xi` 计算出来的移动位数，有可能是负数，比如：主串是 `AAAAAAAAAAAA` 模式串是 `BAA`。 不但不会向后滑动，还有可能倒退。

#### 好后缀规则

好后缀规则和坏字符规则类似，同样是从模式串的末尾开始匹配，当模式串滑动到如图的情况的时候，最后两个字符是匹配的，倒数第三个字符发生了不匹配的情况。

![image-20210312092043100](https://img.varsion.cn/blog-img/20210312092043.png)

这个时候可以使用坏字符规则，根据 `D` 来计算滑动距离。也可以使用好后缀来处理。

我们把已经匹配好的 `AC` 记做 `{U}` 那他在模式串中进行寻找。如果找到了另一个和 `{U}` 匹配的子串 `{U*}` ， 就将模式串滑动到与主串中 `{U}` 对齐的位置。

![image-20210312095952985](https://img.varsion.cn/blog-img/20210312095953.png)

如果在模式串中找不到另一个等于 `{u}` 的子串，我们就直接将模式串，滑动到主串中 `{u}` 的后面，因为之前的任何一次往后滑动，都没有匹配主串中 `{u}` 的情况。



可以分别计算好后缀和坏字符往后滑动的位数，然后取两个数中最大的，作为模式串往后滑动的位数。

### 实现

可以将模式串中的每个字符及其下标都存到散列表中。这样就可以快速找到坏字符在模式串的位置下标了。

关于这个散列表，我只实现一种最简单的情况，假设字符串的字符集不是很大，每个字符长度是 1 字节，用大小为 256 的数组，来记录每个字符在模式串中出现的位置。数组的下标对应字符的 ASCII 码值，数组中存储这个字符在模式串中出现的位置。

```ruby
module Bm
	SIZE = 256
	# String a 表示主串 
	# String b 表示 模式串
	def self.self a, b
		al = a.length
		bl = b.length

		# 构建坏字符哈希表
		bc = generateBC(b, bl)

		suffix, prefix = generateGS(b, bl)
		i = 0
		while i <= al - bl
			j = bl - 1
			while j >= 0
				break if a[i+j] != b[j]
				j-=1
			end
			return i if j < 0 # 匹配成功则返回第一个匹配的字符的位置
			x = j - bc[a[i+j].bytes[0]]
			y = 0
			y = moveByGS(j, bl, suffix, prefix) if j < bl - 1
			i = i + max(x, y)
		end
		return -1
	end

	# 构建坏字符哈希表
	def self.generateBC b, bl
		con = Array.new(SIZE, -1)
		(0...bl).each do |i|
			acc = b[i].bytes[0]
			con[acc] = i
		end
		con
	end

	def self.generateGS b, bl
		suffix = Array.new(bl, -1)
		prefix = Array.new(bl, false)
		(0...bl-1).each do |i|
			j = i
			k = 0
			while j >= 0 && b[j] == b[bl-1-k]
				j-=1
				k+=1
				suffix[k] = j+1
			end
			prefix[k] == true if j == -1
		end
		return suffix, prefix
	end

	def self.moveByGS j, m, suffix, prefix
		k = m - 1 - j
		return j - suffix[k] + 1 if suffix[k] != -1
		r = j + 2
		while r <= m - 1
			return r if prefix[m-r]
		end
		return m
	end

	def self.max m, n
		return m if m > n
		return n if n > m
	end

end
```

## KMP算法

KMP算法和上面的BM算法十分相近。

在主串和模式串匹配过程中，当遇到不可匹配的字符的时候，希望能找到一些瑰丽，可以将模式串多向后滑动几位，从而跳过一些不会匹配的情况。

在模式串和主串匹配的过程中，把不能匹配的那个字符仍然叫作坏字符，把已经匹配的那段字符串叫作好前缀。

![image-20210312103022175](https://img.varsion.cn/blog-img/20210312103022.png)

当遇到坏字符的时候，就要把模式串往后滑动，在滑动的过程中，只要模式串和好前缀有上下重合，前面几个字符的比较，就相当于拿好前缀的后缀子串，跟模式串的前缀子串在比较。

KMP 算法就是在试图寻找一种规律：在模式串和主串匹配的过程中，当遇到坏字符后，对于已经比对过的好前缀，能否找到一种规律，将模式串一次性滑动很多位？

KMP算法也可以提前构建一个数组，用来存放模式串中的每个前缀（这些前缀都有可能是好前缀）的最长可匹配前缀子串。



```ruby
module Kmp
	def self.self a, b
		al = a.length
		bl = b.length
		ne = getNexts(b, bl)

		j = 0

		(0...al).each do |i|
			while j>0 && a[i] != b[j]
				j = ne[j-1] + 1
			end
			j+=1 if a[i] == b[j]

			return i - bl + 1 if j == bl
		end
		return -1
	end

	def self.getNexts b, bl
		ne = Array.new(bl)
		ne[0] = -1
		k = -1
		(1...bl).each do |i|
			while k != -1 && b[k+1] != b[i]
				k = ne[k]
			end
			k+=1 if b[k+1] == b[i]
			ne[i] = k
		end
		return ne
	end
end
```

KMP 算法只需要一个额外的 next 数组，数组的大小跟模式串相同。所以空间复杂度是 O(m)，m 表示模式串的长度。

时间复杂度是 O(m+n)。

