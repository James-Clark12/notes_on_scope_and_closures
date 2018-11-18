# notes_on_scope_and_closures

Chapter 1 - What is Scope

Code typically undergoes three steps before it is executed. These three steps are called compilation and involve:

1. Tokenizing/ Lexing - this is breaking a string into meanngful tokens for the language. E.g. var a = 10, might be converted into: a, =, 10, var etc.

2. Parsing - this takes the stream of toekns and represents their grammatical structure in an Abstract Syntax tree

3. Code Generation - the process of an the AST and turning it into executable code.

(vast simplification obviously).

In JS the compilation often occurs microseconds befor ethe code is executed so there is a focus on performance for quick compilation.

Simpson characterises scope as aconversation between three characters carrying out the compilation process:

1. The Engine - responsible for start to finish compilation and execution of the JS porgram
2. The Copiler - handles the dirty work of parsing and code-generation
3. Scope - collects and maintains a look up list of all the declared variables and enforces a strict set of rules as to how these are accesible to currently executing code.

The conversation for code: var a = 2;

The compiler takes the program tokenizes it and parses it into a tree.

Next, after encountering var a, the copiler asks Scope to see if a variable a already exists for that particular scope collection. If so COmpiler ignores the declaration and moves on. Otherwise, Compiler asks Scope to declare a new varaible called a for that scope collection.

Next Compuler produces produces the code for engine to later execute. To handle the a =2 assignment, the enginer will firrst ask scope if there is a variable called a accessible in the current scope collection. If there is, it will use that, else it will look outside the current scope.

To summarize: two distinct actions are taken for a variable assignment: First, Compiler declares a variable (if not previously declared in the current scope), and second, when executing, Engine looks up the variable in Scope and assigns to it, if found.

When the engine asks scope to look up a varaible like a, there are two types of lookup. Te type is important as it affects the outcome of the lookup.

The two types are: Left hand assignment and right side assignment.

LHS look-up is done when a variable appears on the left-hand side of an assignment operation, and an RHS look-up is done when a variable appears on the right-hand side of an assignment operation.

More precisely, an RHS look-up is indistinguishable, for our purposes, from simply a look-up of the value of some variable, whereas the LHS look-up is trying to find the variable container itself, so that it can assign. In this way, RHS doesn't really mean "right-hand side of an assignment" per se, it just, more accurately, means "not left-hand side".

For the program: console.log(a)

The reference to a is an RHS reference, because nothing is being assigned to a here. Instead, we're looking-up to retrieve the value of a, so that the value can be passed to console.log(..).

By contrast:

a = 2;

The reference to a here is an LHS reference, because we don't actually care what the current value is, we simply want to find the variable as a target for the = 2 assignment operation.

Note: LHS and RHS meaning "left/right-hand side of an assignment" doesn't necessarily literally mean "left/right side of the = assignment operator". There are several other ways that assignments happen, and so it's better to conceptually think about it as: "who's the target of the assignment (LHS)" and "who's the source of the assignment (RHS)".

///// HIS LONG EXAMPLE /////

Consider this program, which has both LHS and RHS references:

function foo(a) {
	console.log( a ); // 2
}

foo( 2 );
The last line that invokes foo(..) as a function call requires an RHS reference to foo, meaning, "go look-up the value of foo, and give it to me." Moreover, (..) means the value of foo should be executed, so it'd better actually be a function!

There's a subtle but important assignment here. Did you spot it?

You may have missed the implied a = 2 in this code snippet. It happens when the value 2 is passed as an argument to the foo(..) function, in which case the 2 value is assigned to the parameter a. To (implicitly) assign to parameter a, an LHS look-up is performed.

There's also an RHS reference for the value of a, and that resulting value is passed to console.log(..). console.log(..) needs a reference to execute. It's an RHS look-up for the console object, then a property-resolution occurs to see if it has a method called log.

Finally, we can conceptualize that there's an LHS/RHS exchange of passing the value 2 (by way of variable a's RHS look-up) into log(..). Inside of the native implementation of log(..), we can assume it has parameters, the first of which (perhaps called arg1) has an LHS reference look-up, before assigning 2 to it.

Note: You might be tempted to conceptualize the function declaration function foo(a) {... as a normal variable declaration and assignment, such as var foo and foo = function(a){.... In so doing, it would be tempting to think of this function declaration as involving an LHS look-up.

However, the subtle but important difference is that Compiler handles both the declaration and the value definition during code-generation, such that when Engine is executing code, there's no processing necessary to "assign" a function value to foo. Thus, it's not really appropriate to think of a function declaration as an LHS look-up assignment in the way we're discussing them here.

Engine/Scope Conversation
function foo(a) {
	console.log( a ); // 2
}

foo( 2 );
Let's imagine the above exchange (which processes this code snippet) as a conversation. The conversation would go a little something like this:

Engine: Hey Scope, I have an RHS reference for foo. Ever heard of it?

Scope: Why yes, I have. Compiler declared it just a second ago. He's a function. Here you go.

Engine: Great, thanks! OK, I'm executing foo.

Engine: Hey, Scope, I've got an LHS reference for a, ever heard of it?

Scope: Why yes, I have. Compiler declared it as a formal parameter to foo just recently. Here you go.

Engine: Helpful as always, Scope. Thanks again. Now, time to assign 2 to a.

Engine: Hey, Scope, sorry to bother you again. I need an RHS look-up for console. Ever heard of it?

Scope: No problem, Engine, this is what I do all day. Yes, I've got console. He's built-in. Here ya go.

Engine: Perfect. Looking up log(..). OK, great, it's a function.

Engine: Yo, Scope. Can you help me out with an RHS reference to a. I think I remember it, but just want to double-check.

Scope: You're right, Engine. Same guy, hasn't changed. Here ya go.

Engine: Cool. Passing the value of a, which is 2, into log(..).

...

////////////////////////////////////////////

If a variable cannot be found in the immediate scope, Engine consults the next outer containing scope, continuing until found or until the outermost (aka, global) scope has been reached.

The simple rules for traversing nested Scope: Engine starts at the currently executing Scope, looks for the variable there, then if not found, keeps going up one level, and so on. If the outermost global scope is reached, the search stops, whether it finds the variable or not.

In conversation form again:

So, revisiting the conversations between Engine and Scope, we'd overhear:

Engine: "Hey, Scope of foo, ever heard of b? Got an RHS reference for it."

Scope: "Nope, never heard of it. Go fish."

Engine: "Hey, Scope outside of foo, oh you're the global Scope, ok cool. Ever heard of b? Got an RHS reference for it."

Scope: "Yep, sure have. Here ya go."

////////////////////////////////////////////

LHS and RHS behave different when no reference can be found at any level of scope.

If an RHS look-up fails to ever find a variable, anywhere in the nested Scopes, this results in a ReferenceError being thrown by the Engine. It's important to note that the error is of the type ReferenceError.

By contrast, if the Engine is performing an LHS look-up and arrives at the top floor (global Scope) without finding it, and if the program is not running in "Strict Mode" [^note-strictmode], then the global Scope will create a new variable of that name in the global scope, and hand it back to Engine.

"No, there wasn't one before, but I was helpful and created one for you."

"Strict Mode" [^note-strictmode], which was added in ES5, has a number of different behaviors from normal/relaxed/lazy mode. One such behavior is that it disallows the automatic/implicit global variable creation. In that case, there would be no global Scope'd variable to hand back from an LHS look-up, and Engine would throw a ReferenceError similarly to the RHS case.

ReferenceError is Scope resolution-failure related, whereas TypeError implies that Scope resolution was successful, but that there was an illegal/impossible action attempted against the result.

Chapter 2 - Lexical Scope - (lexical definition - relating to the words or vocabulary of a language

Not sure why this is a different chapter - just the same as the previous one. Simply outlines how variables first access their closest scope before going beyond them.

Lexical scope is deifned at the Lexical/ Tokenizing state of compilation. 

scope consists of a series of "bubbles" that each act as a container or bucket, in which identifiers (variables, functions) are declared. These bubbles nest neatly inside each other, and this nesting is defined at author-time

No matter where a function is invoked from, or even how it is invoked, its lexical scope is only defined by where the function was declared.

Chapter 3 - Function vs. Block Scopre

"Principle of Least Privilege" [^note-leastprivilege], also sometimes called "Least Authority" or "Least Exposure". This principle states that in the design of software, such as the API for a module/object, you should expose only what is minimally necessary, and "hide" everything else.

It can be a good idea to wrap code in a function to insulate its scope.

However the function name itself still polutes the scope/ namespace and the function must be manually executed in the exterior scope.

TO answer the second issue there is the IIFE pattern:

var a = 2;

(function foo(){

	var a = 3;
	console.log( a ); // 3

})();

console.log( a ); // 2

It allows a function to be executed immediately.

As to the first issue ES6 introduced block scope for let and const.

The let keyword attaches the variable declaration to the scope of whatever block (commonly a { .. } pair) it's contained in. In other words, let implicitly hijacks any block's scope for its variable declaration.

declarations made with let will not hoist to the entire scope of the block they appear in. Such declarations will not observably "exist" in the block until the declaration statement.


Declaring explicit blocks for variables to locally bind to is a powerful tool that you can add to your code toolbox.

let Loops
A particular case where let shines is in the for-loop case as we discussed previously.

for (let i=0; i<10; i++) {
	console.log( i );
}

console.log( i ); // ReferenceError
Not only does let in the for-loop header bind the i to the for-loop body, but in fact, it re-binds it to each iteration of the loop, making sure to re-assign it the value from the end of the previous loop iteration.

Here's another way of illustrating the per-iteration binding behavior that occurs:

{
	let j;
	for (j=0; j<10; j++) {
		let i = j; // re-bound for each iteration!
		console.log( i );
	}
}

Chapter 4 - Hoisting

Both function scope and block scope behave by the same rules in this regard: any variable declared within a scope is attached to that scope.

But there's a subtle detail of how scope attachment works with declarations that appear in various locations within a scope, and that detail is what we will examine here.

the Engine actually will compile your JavaScript code before it interprets it. Part of the compilation phase was to find and associate all declarations with their appropriate scopes. 

When you see var a = 2;, you probably think of that as one statement. But JavaScript actually thinks of it as two statements: var a; and a = 2;. The first statement, the declaration, is processed during the compilation phase. The second statement, the assignment, is left in place for the execution phase.

So, one way of thinking, sort of metaphorically, about this process, is that variable and function declarations are "moved" from where they appear in the flow of the code to the top of the code. This gives rise to the name "Hoisting".

 hoisting is per-scope. 
 
 Function declarations are hoisted. But function expressions are not.
 
 foo();

function foo() {
	console.log( a ); // undefined

	var a = 2;
}

Leads to....

foo(); // not ReferenceError, but TypeError!

Both function declarations and variable declarations are hoisted. But a subtle detail (that can show up in code with multiple "duplicate" declarations) is that functions are hoisted first, and then variables.

While multiple/duplicate var declarations are effectively ignored, subsequent function declarations do override previous ones.

Chapter 5 - Scope closures

 



