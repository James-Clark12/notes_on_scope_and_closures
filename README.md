# notes_on_scope_and_closures

Chapter 1

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


