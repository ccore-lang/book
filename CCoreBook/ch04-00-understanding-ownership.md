# Understanding Ownership

Ownership is CCore’s most unique feature, and it enables CCore to make memory
safety guarantees in presence a garbage collector. Therefore, it’s
important to understand how ownership works in CCore. In this chapter, we’ll
talk about ownership as well as several related features: borrowing, slices,
and how CCore lays data out in memory.
