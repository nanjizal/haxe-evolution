# Early Block

* Proposal: [HXP-NNNN](0000-early-block.md)
* Author: [Justin L Mills](https://github.com/nanjizal)

## Introduction

Early break/return from Blocks.

## Motivation

When using a ```while``` to simulate complex loops from other languages it can be clean to scope loop logic viables, hosting the ```while``` within a block.
To enable all viables related to the while loop logic, remain locally contained like the ```i``` used in a ```for(i in iterator)```.

My revelaton was the limitation of current blocks - you can't break or return early even though blocks are currently an expression.
```Haxe
// valid haxe 
var i: Int = { 1 };
```
Requirements
1) To provided an 'expression return' / 'loop-block break' to allow ```if``` logic to finish the block early.
2) What seemed ideal was to have blocks that allowed ```break``` / ```continue``` transparency to a loop the block was hosted within, but retains local scope.

Pseudocode showing the first feature
```Haxe
var maxLength = 100;
@:early
var blockTriangles: Int = {
   if(length > maxLength) early 0;
   trace('not too large');
   if(!visible) early -1;
   trace('can be seen');
   var tri: Int = this.drawShape();
   tri;
}
trace( blockTriangles );
```
The full motivation extends to allowing inline to return early and to reduce the complexity of reading long if statements.


## Detailed design

Rudy implemented a macro that provides 1) even with some nesting.
```Haxe
class Test {
   static function main() {
      var c = 'b';
      var d = 1;
      var f;
      var a:Int = Macro.early({
			    if (c == 'a') return 1;
          var e:Int = Macro.early({
    		     return 1;
          });
          f = e;
          d = 2;
          if (c == 'b') return 2;
			    if (c == 'c') return 3;
          return 5;
       });
		   trace(a);
		   trace(d);
       trace(f);
   }
}
```
  
```Haxe
class Macro {
	public static macro function early(e) {
		return macro @:pos(e.pos) inline(function() $e)();
	}
}
```
But it is limited as putting an early block within a loop does not allow it to use ```break``` or ```continue```.
For emulating Go language switch structures that require breaks, early blocks are ideal, but the macro above stops use of continue if the simulated switch is within a loop. 

## Impact on existing code

Macro's use blocks a lot so there maybe some complex side effects if not carefull managed or limited.  
With inline code early returns may no longer be a limitation.
```Haxe
@:early
inline function printLots(val: Dynamic, time: Int): Null<Int> {
   if(time == 1000) early null;
   trace( val );
   time+=1;
   return time;
}
```
Early blocks allow code to be cleaner when using complex long if else statments, and empower blocks as a real tool in Haxe and elevate them to status within some other programming languages.

## Drawbacks

May break macros.

## Alternatives

The alternative is ugly inline complex if statements, and headaches trying to fully implement a macro that does not have runtime cost.

## Opening possibilities

Could allow rules on inlines to become more relaxed.

## Unresolved questions

Currently the suggested grammer and the keyword 'early' is arbitary and the implementation may vary.


