---
title: Unit testing
date: 2015-06-09
description:
---

bitwise in javascript with few examples:

1) nqueens

var findQueenSolutions = function(n) {
  var count = 0;
  var all = parseInt(Array(n+1).join('1'), 2);

  (function attempt(ld, cols, rd) {
    var bit;
    var poss = ~( ld | cols | rd ) & all; 

    while( poss > 0 ) {
      bit = poss & -poss; 
      poss -= bit;
      attempt((ld|bit) << 1, (cols|bit), (rd|bit) >> 1);
    }
    
    if (cols === all) {
      count++;
    };
  }(0, 0, 0));

  return count;
};

http://www.devarticles.com/c/a/Cplusplus/Bitwise-Operators/2/