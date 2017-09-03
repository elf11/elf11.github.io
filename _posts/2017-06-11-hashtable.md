---
layout: post
title: Open-addressed double hashed hash table
description: "how to write a hash table in C"
<!-- modified: 2015-09-10 -->
tags: [C, hash table, programming, tutorial]

comments: true
share: true
---

While I was still in my undergrad program at the University Politehnica of Bucharest, great place btw - you should check out their courses (https://ocw.cs.pub.ro), I took some low level programming classes, and one of the first classes that was actually raising some difficulties was the Operating Systems one. The first assignment for that class was to write a multi-platform hash table, meaning your C code had to run on both Windows and Linux platforms. I am not going to get into details about how you make that work, or why is important now, but I was thinking about why did we get a hash table as a first assignment and about the importance of hash tables in computing in general. So I came up with the idea of writing a tutorial about how to write a hash table in C. What you will get from this is a deeper understanding of how the data structure works and how and when to use it, why sometimes it's great to use hash tables and other times it's not.

## What is a hash table?

Suppose we want to design a system for storing car license plate numbers and the name of the person whose name the car is registered in. And we want following queries to be performed efficiently:

1. Insert a license plate number and corresponding information.
2. Search a license plate number and fetch the information.
3. Delete a license plate number and related information.

We could do this in a few different ways, we could store the data using:

1. Array of license plate number and corresponding info (this solution is not really working, we are going to search in linear fashion, and if our data is really big that is going to take time. Even if we keep the license plate numbers - made out of letters and numbers - in a sorted fashion the fastest we can search this structure is in O(log n) time using binary search. We can do better. )
2. A linked list storing license plate number and corresponding info (this solution has the same limitations as the previous one)
3. A direct access table (we create a big array and use the license plate numbers as index in the array will not work always, especially if we have again a lot of data, the extra space required for this will be proportional to the number of entries. But this solution is best when it comes to time complexity, the search time for this is actually O(1) - an entry in the access table is NULL if the license plate is not present, otherwise it stores a pointer to the person whose name the car is registered in)

So we want to be able to search in O(1) time, but without the drawbacks that a direct access table gives us. A hash table is an improvement over the direct access table and consists of an array of "buckets", each bucket stores a key-value pair. The idea is to use a hash function that converts a given license plate or any other key to a smaller number and uses this smaller number as index in a table called hash table. The process is reversible, if we want to get back the initial key-value pair (in our case the license plate number and the person who has that license registered) we give the key to the same hashing function, get back the index and use that to find the value in the array.

### hash function

A hash function converts a license plate number to a small index-integer value that will be used in the hash table as identifier for the key-value pair.A good hash function should have following properties
1. Efficiently computable.
2. Should uniformly distribute the keys (each table position equally likely for each key)

### collision handling

The hash gets as input a key and returns a small index number, so there is a possibility that two keys result in the same value. When we want to insert a new key that maps to an already occupied slot in the hash table we have a collision and we have to fix it (there would be no use to our hash table if we end up having an array of key-value pairs that we need to traverse for each index in the hash table).

Open addressing is a method for handling collisions. In Open Addressing, all elements are stored in the hash table itself. So at any point, size of table must be greater than or equal to total number of keys. Open addressing in a hash table can be done in one of the following 3 ways:

1. linear probing - we look linearly for the next free slot where we can insert another key:value pair, this is a consequence of the hash function that usually looks like this, where S is the size of the table, and hash(x) is the index computed by the hash function for key k
{% highlight bash %}
If slot hash(k) % S is full, then we try (hash(k) + 1) % S
If (hash(k) + 1) % S is also full, then we try (hash(k) + 2) % S
If (hash(k) + 2) % S is also full, then we try (hash(k) + 3) % S 
{% endhighlight %}

The problem of this solution is clustering, which makes searching and finding an empty slot slower.
2. quadratic probing - looks for i*i-th slot in the i iteration
3. double hashing - we use another hash function hash2(k) and we look for i*hash2(k) in the i-th iteration
{% highlight bash %}
If slot hash(k) % S is full, then we try (hash(k) + 1*hash2(k)) % S
If (hash(k) + 1*hash2(k)) % S is also full, then we try (hash(k) + 2*hash2(k)) % S
If (hash(k) + 2*hash2(k)) % S is also full, then we try (hash(k) + 3*hash2(k)) % S 
{% endhighlight %}	


### hash table operations

The operations that our hash table needs to support are the following:

1. Insert(hT, key): insert a new pair key:value in hash table hT

2. Search(hT, key): return the value associated with key from the hash table

3. Delete(hT, key): delete the pair associated with key. This operation is interesting as in if we simply delete a key then search may fail, so that's why slots of deleted keys are marked as "deleted". Insert can insert an item in a deleted slot, but search doesn't look at those slots.

## Hash table implementation

We first define the structures that are going to hold the hash table data, as well as the DELETED element that is going to mark the fact that one element from the hash table has been deleted.

{% highlight c %}
// define a hash table element that will store key-value pairs
typedef struct elem {
    char *key;
    char *val;
} elem;

// the actual hash table stores an array of pointers to elements
// the size of the hash table as well as the current number of elements
typedef struct hTable {
    int size;
    int count;
    elem **elements;
} hTable;

// the deleted item that will mark the slots that will have
// elements that have been deleted
elem DELETED = {NULL, NULL};
{% endhighlight %}


Now we need functions to allocate memory for each element of the hash table as well as for the hash table itself, we also need to be able to free this memory.

{% highlight c %}
// function that allocates memory for each element of the hash table
// and copies the strings key and value in the newly allocated element
static elem *new_elem(const char *key, const char *val) {
    elem *el = malloc(sizeof(elem));
    
    //if (el == NULL)
    //    return NULL;

    el->key = strdup(key);
    el->val = strdup(val);
    //strcpy(el->key,lkey);
    //strcpy(el->val,val);
    return el;
}

// we need to initialize the hash table structure, we will initially have the size
// of the table be equal to 10 and initialize it with calloc (which fills the allocated
// memory with NULL bytes)
hTable *new_table(const int size) {
    hTable *hT = malloc(sizeof(hTable));

    //if (hT == NULL)
    //    return NULL;

    hT->size = size;
    hT->count = 0; // initially there are no elements in the hash table
    hT->elements = calloc((size_t)hT->size, sizeof(elem*));

    //if (hT->elements ==  NULL)
    //    return NULL;
    
    return hT;
}

// we need to be able to delete an element from the hash table
// using the index returned by the hash function
static void del_elem(elem *e) {
    free(e->key);
    free(e->val);
    free(e);
}

//we also need to be able to delete the whole hash table
//to do that we first must make sure that there are no
//more allocated elements in the table and then delete the whole structure
void del_table(hTable *hT) {
    int i;

    for (i = 0; i < hT->size; i++) {
        elem *el = hT->elements[i];
        if (el != NULL) {
            del_elem(el);
        }
    }

    free(hT->elements);
    free(hT);
}
{% endhighlight %}

Next the hash function and how we are implementing the aspects that we talked about before.

{% highlight c %}
/**
 * As we talked before a good hash function should uniformly distribute the keys and
 * should be efficiently computable. For the purpose of this tutorial we are going to
 * use a generic string function: takes as input a string, converts it to an integer,
 * reduces the size of the integer by taking the remainder of the integer with the
 * size of the desired number of buckets
 * We want the data to be evenly distributed, that's not always the case, there are 
 * sets of inputs that always hash to the same value. It is important to know those sets
 * of inputs for a particular hash function, they posses a security issue, for example a malicious
 * user can feed the table a set of colliding keys, making the search linear time O(n) instead of constant time O(1). 
 */
static int hash(const char *key, const int a, const int buckets) {
    long hash = 0;
    const int len = strlen(key);
    int i;


    for (i = 0; i < len; i++) {
        hash = hash + (long)pow(a, len - (i + 1)) * key[i];
        hash = hash % buckets;
    }

    return (int)hash;
}

/**
 * As we talked before the open addressing method to handle hash collision we are going
 * to implement is the double hashing method. As explained we are going to use a second
 * hash function and implement two iterations of the hash function and return that
 * as our hash
 */
static int hash2(const char *key, const int buckets, const int i) {

    int hash0 = hash(key, PRIME_1, buckets);
    int hash1 = hash(key, PRIME_2, buckets);

    return (hash0 + i*(hash1 + 1))%buckets;
}
{% endhighlight %}

Finally we can write the functions for the operations that our hash table is going to perform: insert, search and delete element.

{% highlight c %}
void insert(hTable *hT, const char *key, const char *val) {
    elem *el = new_elem(key, val);

    // cycle through the buckets until we find an empty or deleted one
    int index = hash2(el->key, hT->size, 0);
    elem *cur = hT->elements[index];
    int i = 1;

    while (cur != NULL && cur != &DELETED) {
        if (strcmp(cur->key, key) == 0) {
            del_elem(cur);
            hT->elements[index] = el;
            return;
        }

        index = hash2(el->key, hT->size, i);
        cur = hT->elements[index];
        i++;
    }

    // we found an index that points to a free bucket
    hT->elements[index] = el;
    hT->count++;
}


/**
 * The search function returns the value associated with the key
 * or NULL if the key can not be found in the table
 */
char *search(hTable *hT, const char *key) {
    int index = hash2(key, hT->size, 0);

    elem *el = hT->elements[index];
    int i = 1;

    while (el != NULL && el != &DELETED) {
        if (strcmp(el->key, key) == 0) {
            return el->val;
        }

        index = hash2(key, hT->size, i);
        el = hT->elements[index];
        i++;
    }

    return NULL;
}

void delete(hTable *hT, const char *key) {
    int index = hash2(key, hT->size, 0);
    
    elem *el = hT->elements[index];
    int i = 1;

    while (el != NULL && el != &DELETED) {
        if (strcmp(el->key, key) == 0) {
            del_elem(el);
            hT->elements[index] = &DELETED;
        }

        index = hash2(key, hT->size, i);
        el = hT->elements[index];
        i++;
    }

    hT->count--;
}
{% endhighlight %}


## Conclusion

In conclusion there are advantages and disadvantages of using Hash Tables over other data structures. Some of the advantages are:

1. synchronization
2. in many situations hash tables turn out to be more efficient than search trees or any other table lookup structure. They are widely used in many places, but they are also vulnerable to collisions and that becomes particularly important when hash tables are used in DNS or other web services that can be attacked with denial of services.

The main disadvantages of hash tables are:

1. hash collisions are practically unavoidable when hashing a random subset of a large set of possible keys
2. hash tables become quite inefficient when there are many collisions 

