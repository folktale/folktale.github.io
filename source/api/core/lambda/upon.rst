******************
Function: ``upon``
******************

.. apifunction:: core.lambda.upon

Examples
--------

::

    // Sorting an array of pairs by the first component
    var curry = require('core.lambda').curry
    
    var xss = [[1, 2], [3, 1], [-2, 4]]
    
    function compare(a, b) {
      return a < b?     -1
      :      a === b?    0
      :      /* a> b */  1
    }
    
    function first(xs) {
      return xs[0]
    }
    
    function sortBy(f, xs) {
      return xs.slice().sort(f)
    }
    
    var compareC = curry(2, compare)
        
    sortBy(upon(compareC, first), xss)  // => [[-2, 4], [1, 2], [3, 1]]
