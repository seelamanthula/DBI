# DBI

## Dabase Management System

Extended a Java-based database management system by implementing the core modules required to access data stored on disk. 
wrote code for those classes. And the written code is evaluated by running a set of system tests written using JUnit.

Implemented Tuples, Fields, Catalog, BufferPool, Filer, Aggregates, HeapPage and HeapFile access mode.

### Tuples and Fields: 
          Each Tuple in SimpleDB consists of a collection of Field objects. Field is an interface that different data types (e.g., integer, 
      string) implement. Tuples also have a type (or schema), called a tuple descriptor, represented by a TupleDesc object. 
          This object consists of a collection of Type objects, one per field in the tuple, each of which describes the type of the 
      corresponding field.

### Catalog
          The catalog maintains the tables and schemas of the tables that are currently in the database. 
          Associated with each table is a TupleDesc object that allows operators to determine the types and number of fields in a table.

### BufferPool
          The buffer pool is responsible for caching pages in memory that have been recently read from disk. All operators read and write 
      pages from various files on disk through this buffer pool.
          The buffer pool consists of a fixed number of pages, defined by the numPages parameter to the BufferPool constructor. 
          The BufferPool needs to keep/maintain up to numPages pages using the pages member variable. The type of this pages member variable 
      is Hashtable<PageId, Page>. 

### HeapPage access method
          Access methods provide a way to read or write data on disk that is arranged in a specific way. Common access methods include heap 
      files (unsorted files of tuples) and B+-trees.

### HeapFile access method
          A HeapFile object is arranged into a set of pages, each of which consists of a fixed number of bytes for storing tuples and a header. 
      The size of each page is defined by the constant BufferPool.PAGE SIZE. 
          Each page in a HeapFile is arranged as a set of slots, each of which can hold one tuple (tuples for a given table in SimpleDB are all 
      of the same size). In addition to these slots, each page has a header that consists of a bitmap with one bit per tuple slot. 
      If the bit corresponding to a particular tuple is 1, it indicates that the tuple is valid; if it is 0, the tuple is invalid 
      (e.g., has been deleted or was never initialized). Pages of a HeapFile object are of the HeapPage type which implements the Page interface. 
          Pages are kept in the buffer pool after being read from disk via the HeapFile class's readPage(PageId pid) method.

          It stores heap files on disk in more or less the same format they are stored in memory.
          Each tuple consists of page data arranged consecutively on disk. Each page consists of one or more bytes representing the header, 
      followed by the "BufferPool.PAGE SIZE - [header size in bytes]" bytes of actual page content. 

      The number of tuples that cannot into a page is defined by the formula:
        tupsPerPage = floor((BufferPool.PAGE SIZE * 8) / (tuple size * 8 + 1)), where tuple size is the size of a tuple in the page in bytes. 
        The idea here is that each tuple requires one additional bit of storage in the header. We compute the number of bits in a page 
        (by mulitplying PAGE SIZE by 8), and divide this quantity by the number of bits in a tuple (including this extra header bit) to
        get the number of tuples per page. The floor operation rounds down to the nearest integer number of tuples.

      Once we know the number of tuples per page, the number of bytes required to store the header is simply: 
        headerBytes = ceiling(tupsPerPage/8), where the ceiling operation rounds up to the nearest integer number of byte.

      The low (least significant) bits of each byte represents the status of the slots that are earlier in the file. Hence, the lowest bit of 
      the first byte represents whether or not the first slot in the page is in use. Also, note that the high-order bits of the last byte may 
      not correspond to a slot that is actually in the file since the number of slots may not be a multiple of 8. 
      (Note: all Java virtual machines are big-endian.)
      
### Filter
         returns only the tuples that satisfy a Predicate specified as part of its constructor.
         In other words, this operator filters out any tuples that do not match the Predicate.

### Aggregate
         Implements basic SQL aggregates with a GROUP BY clause.
         Have five SQL aggregates (COUNT, SUM, AVG, MIN, MAX) and support grouping. Use integer division for computing the average since 
      it only supports integers. It support aggregates over a single field, and grouping by a single field. In order to calculate
      aggregates, Need to use an Aggregator interface which merges a new tuple into the existing calculation of an aggregate. 
      The Aggregator is told during construction what operation it should use for aggregation. Subsequently, the client code should call 
      Aggregator.merge() for every tuple in the child iterator. After all tuples have been merged, the client can retrieve a DbIterator 
      of aggregation results. Each tuple in the result is a pair of the form (groupValue, aggregateValue), unless the value of the group 
      by field was Aggregator.
      NO GROUPING, in which case the result is a single tuple of the form (aggregateValue). Note that the output of an Aggregate operator 
      is an aggregate value of an entire group for each call to next(), and that the aggregate constructor takes the aggregation and 
      grouping fields.
