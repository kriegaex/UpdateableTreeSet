UpdateableTreeSet
=================

The problem
-----------

Java's [`TreeSet`](http://docs.oracle.com/javase/7/docs/api/java/util/TreeSet.html) class maintains a __sorted set__
of elements. This is cool, but it comes with a catch: elements are only sorted into the right places at insertion time,
but the sort order is never checked or updated later if element properties relevant for sorting change. So the only way
to update the sort order of a TreeSet is to temporarily remove a changed element and re-add it. This is tricky though,
because you cannot do it within a loop. Quote from [JDK docs for `Iterator.remove()`]
(http://docs.oracle.com/javase/7/docs/api/java/util/Iterator.html):

> Removes from the underlying collection the last element returned by this iterator (optional operation).
> This method can be called only once per call to next().
> __The behavior of an iterator is unspecified if the underlying collection is modified while the iteration is
> in progress__ in any way other than by calling this method.

I.e. only element removal is possible from inside a loop, and only when using an explicit `Iterator`, not the
syntactically nicer form of:

```java
for (element : myTreeSet) {
    if (condition)
        myTreeSet.remove(element)
}
``` 

It gets even more complicated if you want to update an element by removing and re-adding it. This cannot be done from
within a loop unless you create a copy of the set beforehand and loop over the copy so as not to disturb the iterator.
This is all very tedious, see also the corresponding [discussion on StackOverflow.com]
(http://stackoverflow.com/questions/2579679/maintaining-treeset-sort-as-object-changes-value) which sparked my idea of
implementing `UpdateableTreeSet`. My user name there is also *kriegaex*, by the way.

The solution
------------

`UpdateableTreeSet` extends the JDK class and provides a structured way for keeping the sort order up to date. All the
ugly, tedious details are hidden from you as a user, you just do something like this:

```java
import de.scrum_master.util.UpdateableTreeSet;
import de.scrum_master.util.UpdateableTreeSet.Updateable;

class MyType implements Updateable {
    void update(Object newValue) {
        // Change the receiver's value
    }
}

class MyApplication {
    SortedSet<MyType> mySortedSet = new UpdateableTreeSet<MyType>();

    // Add elements to mySortedSet...

    void modifyElements() {
        // Change or remove elements from inside a loop
        for (MyType element : mySortedSet) {
            if (removeCondition)
                markForRemoval(element);
            if (updateCondition)
                markForUpdate(element, newValue);
        }

        // Trigger bulk update
        mySortedSet.updateAllMarked();
    }
}
``` 

As you can see, all the magic is done by three methods of `UpdateableTreeSet`:
  * `markForUpdate`: mark an element as modified after you have changed its "sort key" property/properties.  
  * `markForRemoval`: mark an element as obsolete so it gets removed later.
  * `updateAllMarked`: update/remove all marked elements in one batch operation.

While you can call `markFor*` from within a loop safely, you should only call `updateAllMarked` after you are done with
iterating over the set. That's all there is to it, it is really straightforward.
