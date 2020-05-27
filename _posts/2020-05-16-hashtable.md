---
layout: post
title: HashTable
categories: [data structure]
tags: [HashTable]
fullview: true
markdown: kramdown

---

HashTable是一种通过将键(key)映射为值(value)从而实现快速查找的数据结构。实现HashTable的方式有很多种。本次介绍2种，并给出第一种的PHP版本实现。

我们使用一个链表构成的数组与一个散列函数来实现散列表。当插入键（字符串或几乎其他所有数据类型）和值时，我们按照如下方法操作。

(1) 首先，计算键的散列值。键的散列值通常为int或者long型。请注意，不同的两个键可以有相同的散列值，因为键的数量是无穷的，而int型的总数是有限的。

(2) 之后，将散列值映射为数组的索引。可以使用类似于hash(key) % array_length的方式完成这一步骤，不同的两个散列值则会被映射到相同的数组索引。

(3) 此数组索引处存储的元素是一系列由键和值为元素组成的链表。请将映射到此索引的键和值存储在这里。由于存在冲突，我们必须使用链表：有可能对于相同的散列值有不同的键，也有可能不同的散列值被映射到了同一个索引。

通过键来获取值则需重复此过程。首先通过键计算散列值，再通过散列值计算索引。之后，查找链表来获取该键所对应的值。

如果冲突发生很多次，最坏情况下的时间复杂度是O(N)，其中N是键的数量。但是，我们通常假设一个不错的实现方式会将冲突数量保持在最低水平，在此情况下，时间复杂度是O(1)。

另一种方法是通过平衡二叉搜索树来实现散列表。该方法的查找时间是O(logN)。该方法的好处是用到的空间可能更少，因为我们不再需要分配一个大数组。还可以按照键的顺序进行迭代访问，在某些时候这样做很有用

我们实现一个PHP版的HashTable, 数据结构`Element`、`HashTable`,散列函数，我们使用`Times33*hash`。
功能：添加(覆盖)、删除、获取，内部隐藏逻辑`rehash`。
`rehash`两种情况，一种是扩容、一种是缩容。基于`isRehash`进行判断。


```php
<?php

class Element
{
    public $key;
    public $value;
    public $hash;

    public function __construct($key, $value, $hash)
    {
        $this->key = $key;
        $this->value = $value;
        $this->hash = $hash;
    }
}

class HashTable
{
    private $item = [];
    public $length;
    public $useElement;

    public function __construct()
    {
        $this->initHashTable();
    }

    public function get($key):?string
    {
        $e = $this->_has($key);
        if ($e instanceof Element) {
            return $e->value;
        }
        return null;
    }

    public function store($key, $value): void
    {
        if ($this->isRehash()) {
            $this->rehash();
        }

        $hash = $this->hash($key);
        $slot = $hash % $this->length;

        $element = $this->_has($key);
        if ($element instanceof Element) {
            if ($element->value !== $value) {
                foreach ($this->item[$slot] as $element) {
                    if ($element->key == $key) {
                        $element->value = $value;
                        return;
                    }
                }
            }
        } else {
            if (empty($this->item[$slot])) {
                $this->item[$slot] = [];
            }
            array_push($this->item[$slot], new Element($key, $value, $hash));
            $this->useElement++;
        }

        return;
    }

    public function del($key):?bool
    {
        if (($e = $this->_has($key)) instanceof Element) {
            $this->_del($e);
            return true;
        } else {
            return true;
        }
    }

    private function _del(Element $delElement)
    {
        foreach ($this->item[$slot = ($delElement->hash % $this->length)] as $k => $e) {
            if ($e->key == $delElement->key) {
                unset($this->item[$slot][$k]);
            }
        }

        $this->useElement--;

        if ($this->isRehash()) {
            $this->rehash();
        }
    }

    public function has($key)
    {
        if ($this->_has($key) instanceof Element) {
            return true;
        } else {
            return false;
        }
    }

    private function _has($key):?Element
    {
        $hash = $this->hash($key);
        $slot = $hash % $this->length;

        if (empty($this->item[$slot])) {
            return null;
        }

        foreach ($this->item[$slot] as $element) {
            if ($element->key == $key) {
                return $element;
            }
        }

        return null;
    }

    private function hash(string $key)
    {
        if (is_numeric($key)) {
            return $key;
        } elseif (is_string($key)) {
            $hash = 5381;
            $l = strlen($key);
            for ($i = 0; $i < $l; $i++) {
                $num = ord($key[$i]);
                $hash += $num + $hash << 5 + $hash;
            }
            return $hash;
        }
    }

    private function initHashTable()
    {
        $this->length = 8;
        $this->useElement = 0;
    }

    private function isRehash():bool
    {
        if ($this->useElement > $this->length
            || ($this->useElement > 8 && 2*$this->useElement < $this->length)) {
            $this->rehash();
            echo 'rehash',PHP_EOL;
            return true;
        } else {
            return false;
        }
    }

    private function rehash()
    {
        if ($this->useElement > $this->length) {
            $newLength = 2 * $this->length;
        } else {
            $newLength = $this->length / 2;
        }

        $newItem = [];

        foreach ($this->item as $slot => $items) {
            foreach ($items as $item) {
                $slot = ($item->hash = $this->hash($item->key)) % $newLength;

                if (empty($newItem[$slot])) {
                    $newItem[$slot] = [];
                }
                array_push($newItem[$slot], $item);
            }
        }

        $this->length = $newLength;
        $this->item = $newItem;
    }

    public function debugHashTable()
    {
        echo json_encode($this->item, 128);
    }

    public static function main() {

        $hashTable = new HashTable();

        $items = [
            'key' => '1212',
            'c' => '12311',
            122 => '1231',
            'key1' => '1212',
            'c1' => '12311',
            1221 => '1231',
            'ke' => '1212',
            'ky1' => '1212',
            'ey1' => '1212',
            'ky' => '1212',
        ];

        foreach ($items as $k => $v) {
            $hashTable->store($k, $v);
        }

        foreach ($items as $k => $v) {
            echo $k, ":", $hashTable->get($k), PHP_EOL;
        }

        echo '添加key后结果:',PHP_EOL;
        $hashTable->debugHashTable();

        foreach ($items as $k => $v) {
            $hashTable->del($k);
            if ($k == 'ey1') {
                break;
            }
        }

        echo '删除key后，结果:',PHP_EOL;
        $hashTable->debugHashTable();
        echo $hashTable->length;
    }
}

HashTable::main();

```

判断一个哈希算法的好坏有以下几点:
* 一致性，等价的键必然产生相等的哈希值；
* 高效性，计算简便；
* 均匀性，均匀地对所有的键进行哈希。


更多延伸阅读
* php HashTable
* redis HashTable
* php vs redis rehash方式为什么不一样
* hash 函数实现方式：Times33(djb2), sdbm,lose
