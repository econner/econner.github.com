---
layout: post
title: Understanding Python Dictionaries
category: writing
---

<span class="meta">March 01, 2014</span>

<style>
h2.sectionTitle {
    margin-bottom: 10px;
}
#post ul {
    margin-left: 20px;
}
</style>

<h2 class="sectionTitle">Understanding Python Dictionaries</h2>

Python implements its dictionary as a hash table that uses open addressing to
resolve collisions.  The hash table occupies a contiguous block of RAM broken up
into slots, each of which stores one and only one entry.  Essentially a python
dictionary is just an array in memory where a hash value of each newly inserted key is
used to find a slot in the array to place the new entry.

<h2 class="sectionTitle">The Hash</h2>

Python lets you use seemingly arbitrary objects as keys to dictionaries by
utilizing the object's ``__hash__`` method:

{% highlight python %}
>>> myobj = object()
>>> mydict = {myobj: 'hello'}
>>> mydict
{<object at 0x102bec920>: 'hello'}
{% endhighlight %}

Any object that defines a ``__hash__`` method is considered hashable and thus
can be used as the key in a dictionary.  ``__hash__`` should return an integer,
which python then uses to find the key a slot in the hash table.
If ``__hash__`` does not return an int python attempts to coerce the value to
an int, which can produce less than desirable effects.

``object`` has a default ``__hash__`` function that returns a value
related to the __identity__ of the object.  In python every object has an identity
which can be observed using the builtin ``id()`` function.  Using the identity
makes sense as it is required to be a unique, constant value for the object during
its lifetime.  The natural choice for identity then is the object's
memory address:

{% highlight python %}
>>> myobj = object()
>>> myobj
<object at 0x106f60770>
>>> id(myobj)
4411754352
>>> int(str('0x106f60770'), 16)
4411754352
{% endhighlight %}

Now, if we take the hash of this object we get,

{% highlight python %}
>>> hash(myobj)
275734647
{% endhighlight %}

which is...some random number.  But, it turns out this seemingly random number is
just the identity of the object divided by 16.

{% highlight python %}
>>> id(myobj) / 16 == hash(myobj)
True
{% endhighlight %}

(I don't know why python divides id by 16 for the hash.)

<h2 class="sectionTitle">The Hash Table</h2>

In CPython, the hash table starts with ``PyDict_MINSIZE`` slots which
as of this writing is 8.  To find slots for newly added items, python uses the
expression ``hash(key) & mask`` where ``mask`` is always equal to the current capacity
of the dictionary (which must be a power of 2) minus 1.  The bitwise representation
of 2^n - 1 is always a sequence of 1's so the mask will zero out all bits except
the lower order n-1 bits.

For example, say we want to insert the key 'abc' with value 5.  We first need to find
an empty slot in the hash table,

{% highlight python %}
>>> hash("abc") & (8 - 1)
3
# http://stackoverflow.com/questions/2267362/
>>> int2base(hash("abc"), 2)
'...0101000010100000100100111101000010100011' # lower 3 bits are '011'
{% endhighlight %}

In the hash table this might look something like:

<table style="width:250px; font-family:monospace; border: 1px solid #ccc; text-align: center;">
    <tr>
        <th>Idx</th>
        <th>Hash</th>
        <th>Key</th>
        <th>Value</th>
    </tr>
    <tr>
        <td>000</td>
        <td></td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>001</td>
        <td></td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>010</td>
        <td></td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>011</td>
        <td>...100<b>011</b></td>
        <td>"abc"</td>
        <td>5</td>
    </tr>
    <tr>
        <td>100</td>
        <td></td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>101</td>
        <td></td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>110</td>
        <td></td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <td>111</td>
        <td></td>
        <td></td>
        <td></td>
    </tr>
</table>

So, what happens when we get a collision?  Python __probes__ to find the next
empty slot.  One way to probe would be to search the slots in order from the
occupied slot until an empty one is found.  Python uses a different form of probing
to find the next available slot.  The exact details are not important, but python's
probing uses the higher order bits of the hash code to
prevent long chains of collisions.
(See [dictobject.c:35-125](http://hg.python.org/cpython/file/52f68c95e025/Objects/dictobject.c)).

The hash table gets resized when the dictionary becomes 2/3rds full.  The size of
the table will be doubled (e.g. to 16) and the lower order 4 bits will then
be used to redistribute keys in the block of memory.





