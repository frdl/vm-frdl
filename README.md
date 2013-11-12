vm-shim
=======

This began as a wan attempt to reproduce/polyfill/infill the Node.JS 
<code>vm#runIn<Some?>Context()</code> methods in browsers. It has transformed 
into this current tan muscular self-assured and smiling project before you.

I'd wanted to show that shimming in the browser really could be done directly, 
partly to avoid  
[vm-browserify](https://github.com/substack/vm-browserify) which uses iframes to 
create and clone globals and contexts, and partly to side-step Node.js's 
*contra-normative* implementations of `runInContext` methods.

It's actually a "why didn't I think of that?" solution to problems such as -

+ why don't vm methods accept *functions* as arguments, not just strings?
+ why don't eval() and Function() accept *functions* as arguments?
+ why do eval() and Function() inexcusably leak un-var'd symbols to the global 
scope, in browser & node.js environments?
+ how can we inspect items defined in closures?
+ how can we override (or mock) them?


methods
-------

+ <code>vm#runInContext(code, context)</code>
+ <code>vm#runInNewContext(code, context)</code>
+ <code>vm#runInThisContext(code)</code>


work-in-progress
----------------

I had been looking for a way to use *reflection* in closures, due to the side-
effects from another exchange of rants with Phil Walton.  But then I realized we 
only need to mock certain items in closures at any time - not just inspect them 
during tests. So I've come up with a scope injection utility for that which 
depends on `runInNewContext`.  This will be added to the vm-shim API when "done"


node tests
----------

Using Misko Hevery's [jasmine-node](https://github.com/mhevery/jasmine-node) to 
run command line tests on node (even though this project initially aimed at a 
browser shim).

The `package.json` file defines three test script commands to run the tests via 
jasmine-node without the browsers:

    npm test 
    # => jasmine-node --verbose ./test/suite.spec.js

    npm run-script test-vm 
    # => jasmine-node --verbose ./test/vm-shim.spec.js
    
    npm run-script test-scope
    # => jasmine-node --verbose ./test/scope.spec.js

browser tests
-------------

Using Toby Ho's MAGNIFICENT [testemjs](https://github.com/airportyh/testem) to drive tests in 
multiple browsers for jasmine-2.0.0 (see how to 
[hack testem for jasmine 2](https://github.com/dfkaye/testem-jasmine2)), as well 
as jasmine-node.  The following command uses a custom launcher for jasmine-node 
in testem:

    testem -l j
  
rawgithub page
--------------

Using @pivotallabs' 
<a href='http://jasmine.github.io/2.0/introduction.html'>jasmine 2.0</a> for the 
browser suite.

The *jasmine2* test page is viewable on 
<a href='//rawgithub.com/dfkaye/vm-shim/master/test/jasmine2-test.html' 
   target='_new' title='opens in new tab or window'>
  rawgithub</a>.
  
  
npm
---

    TODO 
    

implementation
--------------

TODO - update first paragraph -

First-cut implementation is `vm.runInContext(code, context)`. The `Function()` 
constructor is at the core.  The *code* param may be either a string or a 
function.  The *context* param is a simple object with key-value mappings.  For 
any key on the context, a new *var* for that key is prefixed to the code.  The 
code is passed in to `Function()` so that the keynames can't leak outside the 
new function's scope.  [8 Nov 2013] As a final touch, `with()` is used on the 
context so any modifications are captured.

Refactored [8 Nov 2013]: a lot of little things involved - biggest is that 
`runInThisContext` now uses `eval()` internally, and the other two use `with()` 
inside of `Function()`.

[10 Nov] Having discovered that eval() leaks globals (!?!) if symbols are not 
var'd, all methods rely on helper methods to scrape EVERY global added by its 
internal `eval()` (or `Function()`) call.  


example tests
-------------

The unit tests demonstrate how `runInContext` and `runInNewContext` methods work 
by passing a `context` param containing a reference to the test's expectation 
object or function.

Example runInContext test passes the `expect` function via context argument:
    
    it("overrides external scope vars with context attrs", function() {

      var attr = "shouldn't see this";
      
      var context = {
        attr: 'ok', 
        expect: expect  // <-- pass expect here
      };
      
      vm.runInContext(function(){
        expect(attr).toBe('ok');
        expect(attr).not.toBe('should not see this');
      }, context); 
      
    });

Example runInNewContext test to verify context is returned:

    it('should return context object', function () {
      var context = { name: 'test' };
      
      var result = vm.runInNewContext('', context);
      
      expect(result).toBe(context);
      expect(result.name).toBe('test');
    });
    
Example runInThisContext test to verify accidental is not placed on global scope:
    
    it("should not leak accidental (un-var'd) globals", function() {
    
      vm.runInThisContext(function(){
        accidental = 'defined';
      });
      
      expect(global.accidental).not.toBeDefined();
    });

    
first success
-------------
Just noting for the record:

+ Original idea emerged late at night 17 SEPT 2013 
+ First implemented with rawgithub approach 18 SEPT, 
+ Full success including objects as properties of the context argument 19 SEPT.
+ Breaking the usual TDD procedure:
  + Started with console statements and prayer ~ removed both for real unit tests
  + Tape tests added 20 SEPT.
  + Jasmine tests/page added 20 SEPT.
+ Error, and leakage tests added 21 SEPT.
+ runInNewContext, runInThisContext methods added; runInContext refactored 4 OCT 2013
+ CoffeeScript test with jasmine-node added 6 OCT
+ tape test written in CoffeeScript test added 7 OCT
+ scope injection tests started 21 OCT
+ scope injection: spec started, tests updated, testem.json added 6 NOV 2013
+ massive refactoring 8 NOV 2013
  - certain cases were just wrong (needed 'eval()' for 'runInThisContext()', e.g.)
  - new/completed bdd specs for both vm-shim and scope mocking (temp name is 'mock')
+ last global leakage fixed 10 NOV
+ anger and disbelief over blatantly wrong implementations of eval() and 
Function() mounting...

TODO
----
- need rawgithub viewable test page that also works with testem (coming up)
- ready to junk the tape and coffeescript tests
- move `junk-drawer` from afterthought to feature
- npm

  