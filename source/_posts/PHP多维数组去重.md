---
title: PHP多维数组去重
date: 2017-02-15 06:25:18
categories:
- PHP
---
```
/**
 * 多维数组去重
 * @param array 
 * @return array
 */
function super_unique($array)
{
    $result = array_map("unserialize", array_unique(array_map("serialize", $array)));

    foreach ($result as $key => $value)
    {
        if ( is_array($value) ) {
            $result[$key] = super_unique($value);
        }
    }

    return $result;
}
```