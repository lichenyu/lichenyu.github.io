title: Java - Collections示例
date: 2019/03/21
categories:
- Program Development
tags:
- Java
---


```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.HashMap;
import java.util.HashSet;
import java.util.Iterator;
import java.util.List;
import java.util.Map.Entry;

public class CollectionsDemo
{

    public static void main(String[] args)
    {
        ArrayList<Integer> myList = new ArrayList<Integer>();
        HashSet<String> mySet = new HashSet<String>();
        HashMap<String, Integer> myMap = new HashMap<String, Integer>();

        // ---------- adding ----------
        myList.add(2);
        myList.add(1);
        myList.add(3);
        mySet.add("item2");
        mySet.add("item1");
        mySet.add("item3");
        myMap.put("key1", 999);
        myMap.put("key2", 888);
        myMap.put("key3", 777);
        myMap.put("key4", 777);
        myMap.put("key5", 666);
        
        // ---------- searching ----------
        System.out.println(myList.contains(1));
        System.out.println(myList.indexOf(1));
        System.out.println(mySet.contains("item1"));
        System.out.println(myMap.containsKey("key1"));
        
        System.out.println();

        // ---------- removing & iterating ----------
        // arg is idx
        myList.remove(2);
        for(int i = 0; i < myList.size(); ++i)
        {
            System.out.println(myList.get(i));
        }
        myList.add(3);
        for(int i : myList)
        {
            System.out.println(i);
        }
        
        // for iteration delete: record first, delete after iterating
        mySet.remove("item2");
        Iterator<String> it = mySet.iterator();// <---------- iterator
        while(it.hasNext())
        {
            String cur = it.next();
            System.out.println(cur);
        }
        mySet.add("item2");
        for(String s : mySet)// <---------- for each
        {
            System.out.println(s);
        }

        myMap.remove("key1");
        Iterator<Entry<String, Integer>> it2 = myMap.entrySet().iterator();// <---------- iterator
        while(it2.hasNext())
        {
            Entry<String, Integer> entry = it2.next();
            System.out.println("key: " + entry.getKey() + ", value: " + entry.getValue());
        }
        myMap.put("key1", 999);
        for(Entry<String, Integer> entry : myMap.entrySet())// <---------- for each
        {
            System.out.println("key: " + entry.getKey() + ", value: " + entry.getValue());
        }

        System.out.println();

        // ---------- sorting ----------
        Collections.sort(myList);
        for(int i : myList)
        {
            System.out.println(i);
        }
        
        ArrayList<String> mySetList = new ArrayList<String>(mySet);
        Collections.sort(mySetList, (s1, s2) -> s1.compareTo(s2) * -1);// <---------- sort reverse
        for(String s : mySetList)
        {
            System.out.println(s);
        }

        List<Entry<String, Integer>> entryList = new ArrayList<Entry<String, Integer>>(myMap.entrySet());
        Collections.sort(entryList, (en1, en2) -> en1.getKey().compareTo(en2.getKey()) * -1);// <---------- by key
        for(Entry<String, Integer> en : entryList)
        {
            System.out.println("key: " + en.getKey() + ", value: " + en.getValue());
        }
        Collections.sort(entryList, (en1, en2) -> {
            if(!en1.getValue().equals(en2.getValue()))
            {
                return en1.getValue().compareTo(en2.getValue()) * -1;
            }
            else
            {
                return en1.getKey().compareTo(en2.getKey());
            }
        });// <---------- by value
        for(Entry<String, Integer> en : entryList)
        {
            System.out.println("key: " + en.getKey() + ", value: " + en.getValue());
        }
    }
}
```
