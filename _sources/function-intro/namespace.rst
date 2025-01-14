..  Copyright (C)  Dave Parillo.  Permission is granted to copy, distribute
    and/or modify this document under the terms of the GNU Free Documentation
    License, Version 1.3 or any later version published by the Free Software
    Foundation; with Invariant Sections being Forward, and Preface,
    no Front-Cover Texts, and no Back-Cover Texts.  A copy of
    the license is included in the section entitled "GNU Free Documentation
    License".

.. include:: <isonum.txt>

.. index:: namespace

Namespaces
==========
When we introduced functions, we noted that all functions are *by default* global.
Another way of saying this is that they are by default in the *global namespace*.
The ``namespace`` keyword provides a mechanism
to avoid polluting the global namespace with too many names.


A ``namespace`` is simply a named block that defines a scope.
Namespaces provide a method for preventing name conflicts in large projects.

Symbols declared inside a namespace block are placed in a named scope that prevents them from being mistaken for identically-named symbols in other scopes.

Multiple namespace blocks with the same name are allowed. 
All declarations within those blocks are declared in the named scope.

.. code-block:: cpp

   // declare some things in the mesa namespace
   namespace mesa {
     int i = 0;
     double pi = 3.1416;
     void details (char);
   }

   void mesa::details (char c) { // define the function declared earlier
     // do something
   }

   //void mesa::oops () {       // error: oops not yet declared in mesa namespace
   //}

   namespace mesa {  // a separate mesa namespace block
     void oops ();
     namespace cisc {
       double pi = 3.14159265358979;  // not the same variable as mesa::pi
     }
   }

   int main () {
     using mesa::cisc::pi;
     pi = 3.f;
     mesa::details('a');
   }

   

The larger your project, the more important it is to partition the global namespace.
By default, all symbols are declared in the *global namespace* (``::``).

What is the problem with the global namespace?

- There is only 1 of them
- Name conflicts can be common on large projects
- Complicates mixing third party libraries

Well-behaved third party libraries will not put much (if anything) in the global namespace.

You can put anything in a namespace, except ``main``.
The function main has a few special rules and one is that it must be in the global namespace.

The ``using`` directive allows all the names in a namespace to be used 
without the namespace-name as an explicit qualifier. 
Use a using directive in an implementation file (i.e. ``*.cpp``) 
if you are using several different identifiers in a namespace; 
if you are just using one or two identifiers, 
then consider a using declaration to only bring those identifiers into scope 
and not all the identifiers in the namespace. 
If a local variable has the same name as a namespace variable, 
the namespace variable is hidden. 
It is an error to have a namespace variable with the same name as a global variable.

.. index::
   pair: namespace; anonymous

Anonymous namespaces
--------------------
It is possible to define a namespace without a name.
Unnamed namespace members have potential scope 
from their point of declaration to the **end of the translation unit**.

In other words, they behave a bit like global variables, 
visible to all functions, but *only* within the source file
where the namespace is defined.

Unnamed (or anonymous) namespaces are considered a 'modern'
C++ alternative to declaring variables as ``static``
within a translation unit.

.. code-block:: cpp


   static int i;

   // anonymous namespace
   namespace {
     int i;
   }

At one point the C++ standards committee planned to deprecate the use
of ``static`` in this way and force the use of namespaces,
but that decision was reversed in 2009.

Unnamed namespaces are preferred over the use of the keyword static
for several reasons:

- The ``static`` keyword can have different meanings depending on context

  Namespaces only have one purpose: to define and enclose a scope.

  Namespaces provide a uniform and consistent way of controlling visibility.
  You don't have to use different tools for the same thing.

- A namespace can encapsulate anything.
  Only functions and objects can be declared static.

  User defined types, which is the focus of the second half of this course
  cannot be declared static.

  .. code-block:: cpp

     // valid statements
     static void my_function() { /* function body */ }
     static int  my_variable;

     // invalid
     static class sample_class { /* class body */ };
     static struct sample_struct { /* struct body */ };

     // valid
     namespace 
     {  
       class sample_class { /* class body */ };
       struct sample_struct { /* struct body */ };
     }


- When using an anonymous namespace, 
  the function/object name is mangled properly, 
  which allows you to see something like "(unique namespace)::xyz" 
  in the symbol table after de-mangling, 
  and not just "xyz" with static linkage.


.. tabbed:: anon_namespace_tab

   .. tab:: Example

      At compile time,
      This definition is treated as a definition of a namespace with a
      unique name and a using-directive in the current scope.


      .. code-block:: cpp

         namespace {
           int i;       // defines ::(unique)::i
         }

      The unique name of the namespace is hidden.
      Since it is not known, no code outside the current
      translation unit can access it.

      It is technically possible to have more than one unnamed
      namespace in the same translation unit.

      .. code-block:: cpp

         namespace {
           int i;       // defines ::(unique)::i
         }

         namespace A {
             namespace {
                 // reusing the name 'i' in this scope is a bad idea . . .
                 int i; // A::(unique)::i
                 int j; // A::(unique)::j
             }
             void g() { i++; } // A::unique::i++
         }
          
      As a best practice, you should keep unnamed namespaces to a minumum
      and declare them near the top of your translation unit
      so that they stand out.

   .. tab:: Run It

      .. activecode:: ac_anon_namespace_1
         :language: cpp
         :nocodelens:

         #include <iostream>

         namespace {
           int i;       // defines ::(unique)::i
         }

         void f() {
           i++;         // increments ::(unique)::i
         }
          
         namespace A {
           namespace {
             // reusing the name 'i' in this scope is a bad idea . . .
             int i;          // A::(unique)::i
             int j;          // A::(unique)::j
           }
           void g() { i++; } // A::unique::i++
         }
          
         using namespace A;  // introduces all names from A into global namespace

         void h() {
           // error: ::(unique)::i and ::A::(unique)::i are both in scope
           // i++;

           A::i++; // ok, increments ::A::(unique)::i
           j++;    // ok, increments ::A::(unique)::j
           f();
         }

.. index::
   pair: namespace; using directive

Prefer using declarations to ``using namespace std``
----------------------------------------------------
What's wrong with ``using namespace std;``?

Nothing, technically.
It was a simplification in your first semester classes.
The intent was to avoid 'burdening' you with having to care about this technical detail.

**BUT**

The using-directive ``using namespace std;`` at any namespace scope introduces 
*every* name from the namespace std into the global namespace 
(since the global namespace is the nearest namespace that contains both std and any user-declared namespace), 
which may lead to undesirable name collisions. 
This, and other using directives are generally considered bad practice at file scope of a header file.
Additionally, shadowing names in the standard namespace can lead to unexpected behaviors.


.. tabbed:: namespace

   .. tab:: Example: namespace std

      It can be hard to remember every name that might be
      imported when ``using namespace std;``.
      Even when only 1 header is included.

      The following example seems innocent enough,
      until you learn that :io:`showpoint<manip/showpoint>` is a name
      in ``std::ios``

      Run the following example twice,
      first as is, then remove the line ``bool showpoint = true;``:

      .. activecode:: ac_using_namespace_std_1
         :language: cpp
         :nocodelens:

         #include <iostream>

         using namespace std;

         int main () {
           bool showpoint = true; 

           cout << "1.0 with showpoint: " << showpoint << 1.0 << '\n'
                << "1.0 with noshowpoint: " << noshowpoint << 1.0 << '\n';

         }
   
Errors using namespace directives are seldom this obviously wrong.


.. tabbed:: so_question

   .. tab:: Stack Overflow

      Here is a simplification of a 
      `real question <https://stackoverflow.com/questions/2712076/how-to-use-an-iterator>`_
      asked on stack overflow:

      .. activecode:: ac_using_namespace_std_so_2
         :language: cpp
         :compileargs: ['-Wall', '-Wextra', '-std=c++11']

         #include <iostream>
         #include <vector>
         #include <cmath>
         using namespace std;

         struct point {
           float x;
           float y;
         };

         // calculate the distance between two points
         float distance(const point& p1, const point& p2) {
           return sqrt((p1.x - p2.x)*(p1.x - p2.x) +
                       (p1.y - p2.y)*(p1.y - p2.y));
         }

         int main() {
           vector <point> po;
           point p1; p1.x = 0; p1.y = 0;
           point p2; p2.x = 100; p2.y = 100;
           po.push_back(p1);
           po.push_back(p2);

           for (auto ii = po.begin(); ii != po.end(); ii++) {
             for (auto jj = po.begin(); jj != po.end(); jj++) {
               cout << distance(ii,jj) << " ";
             }
           }
           return 0;
         }
             
            
      This code compiles and runs and says the answer is: ``0 1 -1 0``.
      I think we can all agree that is not the correct answer for two points ``(0,0)`` and ``(100,100)``.

.. admonition:: Try This!

   What is wrong with the program?

   Can you fix it?

   **Hint:**

   The problem with each of the preceding programs was that the 
   author did not realize the potential pitfalls with ``using namspace std;``.


.. index::
   pair: Herb Sutter; namespace using
   pair: Andrei Alexandrescu; namespace using

One final word from two experts:

  **Summary**

  Namespace usings are for your convenience, not for you to inflict on others: 
  Never write a ``using`` declaration or a ``using`` directive before an ``#include`` directive.

  Corollary: In header files, 
  don’t write namespace-level using directives or using declarations; 
  instead, explicitly namespace-qualify all names. 
  (The second rule follows from the first, 
  because headers can never know what other header ``#include`` might appear after them.)

  **Discussion**

  In short: 
  You can and should use namespace using declarations and directives liberally 
  in your implementation files after ``#include`` directives and feel good about it. 
  Despite repeated assertions to the contrary, 
  namespace ``using`` declarations and directives are not evil and they do not defeat the purpose of namespaces. 
  Rather, they are what make namespaces usable.

  -- Herb Sutter and Andrei Alexandrescu, C++ Coding Standards


-----

.. admonition:: More to Explore

  - From: cppreference.com: 
    :lang:`namespace declarations <namespace>` and 
    :lang:`namespace aliases <namespace_alias>`

  - From stack overflow:
    `Why is using namespace std considered bad practice? <https://stackoverflow.com/questions/1452721/why-is-using-namespace-std-considered-bad-practice>`_


