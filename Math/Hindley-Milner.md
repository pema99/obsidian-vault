A Hindley-Milner (HM) type system is essentially an extension of typed lambda calculus with parametric polymorphism. This polymorphism is usually implicit and can importantly only be used at the 'top-level'. It has some extremely nice properties such as the type of any term being inferable from its use, and a fairly simple, almost built-in, method for inferring said types.
# Types of types
HM deals with 2 kinds of type: Monotypes (aka. simple types) and polytypes (aka. type schemes).

Monotypes are simple, non-polymorphic types such as $Int$, $Bool$, $List(String)$ and so on. For simplicity, I'll write concrete monotypes with uppercase first letter, and type variables with lowercase.

Polytypes, or type schemes, are universally quantified monotypes, such as $\forall a. List(a)$, which is concretely the type of a generic list, or $\forall a. a \xrightarrow{} a$, which represents the type of the identity function. Polytypes are, as the name implies, polymorphic - the $a$ in those types can be literally any type.
# Instantiation and generalization
We can convert between monotypes and polytypes using _instantiation_ and _generalization_.

Instantiation takes a polytype and returns a monotype by removing the quantifier and replacing all usages of it with some monotype. For example, $\forall a. a \xrightarrow{} a$ could be instantiated to either $Int \xrightarrow{} Int$ or $Bool \xrightarrow{} Bool$.

Generalization takes a monotype and returns a polytype by quantifying over all free type variables in the monotype. For example, $List(a)$ would generalize to $\forall a. List(a)$.

Importantly, the types $a$ and $\forall a. a$ are different. The former represents some _concrete_ type (think $Int$ or $Bool$) which we don't yet know, so must refer to with a variable. The latter represents _literally any_ type, since it is polymorphic.

HM only allows universal quantification of types at the "topmost" layer. IE, any polytype must have the form $\forall [variables]. [type]$, with no nesting of the quantifiers. This, as far as I understand, makes the type system _decidable_, as opposed to the type system dubbed 'System F' which does not have this restriction.
# Reading inference rules
The meat of HM can be specified concisely with a small set of _inference rules_. These look scary, but are fairly straight forward once you understand how to read them. Every inference rule has a set of premises and a conclusion, which are put on the top and bottom of a horizontal bar, like so:
$$
\begin{prooftree}
\AxiomC{P}
\AxiomC{Q}
\BinaryInfC{P $\land$ Q}
\end{prooftree}
$$
Which reads as "if I know P, and I know Q, then I know P and Q". The rule has 2 premises and a single conclusion.

When dealing with actual type-theoretic inference, we often carry around an _environment_, usually written as $\Gamma$. The environment contains all the types of terms we have already inferred. Really, it is just a set of a terms and their corresponding types. The main operation we do on the environment is to make statements like $ \Gamma \vdash x : \tau $ which means "given the information in the environment, we know $x$ is of type $\tau$". This is called a typing judgment.

The '$\vdash$' operator, called the _turnstile_, can be thought of a statement stating that a proof exists. Concretely, $ \Gamma \vdash a $ means that given $\Gamma$, the environment, we can prove $a$, whatever that might be.
# Inference rules for basic HM
Putting the previous knowledge to use, let's look at the inference rules for HM, one by one.
### Variable usage
The first rule describes how to assign types to usages of variables. It is written as:
$$
\begin{prooftree}
\AxiomC{$x : t \in \Gamma$}
\UnaryInfC{$\Gamma \vdash x : t$}
\end{prooftree}
$$
In english: "**If** the environment actually contains the information that $x$ is of type $t$, **then** the environment proves that $x$ is of type $t$.", which frankly is stating the obvious.

Interpretation: The environment keeps track of all the variables we have inferred the type of so far.
### Function application
The next rules tells us how to type usages of functions in applications (function calls):
$$
\begin{prooftree}
\AxiomC{$\Gamma \vdash f : t_1 \xrightarrow{} t_2$}
\AxiomC{$\Gamma \vdash a : t_1 $}
\BinaryInfC{$\Gamma \vdash f(a) : t_2 $}
\end{prooftree}
$$
In english: "**If** the environment proves that $f$ is a function from type $t_1$ to $t_2$, and that $a$ is a expression of type $t_1$, **then** the environment proves that the application $f(a)$ is of type $t_2$". This should be very reminiscent of modus-ponens for those familiar with propositional logic.

Interpretation: Giving the correct type of input to a function should result in the correct type of output.
### Function abstraction
This rule tells us how to type function abstractions (declarations of functions, concretely as lambdas):
$$
\begin{prooftree}
\AxiomC{$\Gamma, x : t_1 \vdash e : t_2$}
\UnaryInfC{$\Gamma \vdash (fun(x) \xrightarrow{} e) : t_1 \xrightarrow{} t_2 $}
\end{prooftree}
$$
In english: "**If** the environment _and_ the assumption that $x$ is an expression of type $t_1$ prove that $e$ is an expression of type $t_2$, **then** the environment proves that the lambda $fun(x) \xrightarrow{} e$ has the type of a function from $t_1$ to $t_2$, written as $t_1 \xrightarrow{} t_2$.

Interpretation: A lambda expression is a function taking an input of some initially unknown type, returning a value of the type we infer body to, and the body may make use of the input.

This rule implies something about how we should infer the type of lambda expression. First, we make up a new, fresh type variable for the input parameter. Then we add info to the environment stating that the input parameter has that new type, and then we infer the type of the lambda body in that altered environment.
### Let bindings
This rule tells us how to type let bindings of the form `let x = e1 in e2` where `x` is an identifier and `e1` and `e2` are both expressions.
$$
\begin{prooftree}
\AxiomC{$\Gamma \vdash e_1 : t_1$}
\AxiomC{$\Gamma, x : t_1 \vdash e_2 : t_2$}
\BinaryInfC{$\Gamma \vdash (let \ x = e_1 \ in \ e_2) : t_2$}
\end{prooftree}
$$
In english: "**If** the environment proves that $e_1$ is an expression of type $t_1$, and the environment _with_ the assumption that $x$ is an expression of type $t_1$ proves that $e_2$ is an expression of type $t_2$, **then** the environment proves that the let binding $let \ x = e_1 \ in \ e_2$ is of type $t_2$."

Interpretation: Let bindings bind a name to a value in a following expression, and evaluate to a value of the type we infer that following expression to be. The following expression can naturally make use of the bound name.

As for lambda expressions, this implies that we need to alter the environment before inferring the type for the following expression.
### If-then-else expression
This rule usually isn't included in the minimal presentation of HM, but I include it since it is useful:
$$
\begin{prooftree}
\AxiomC{$\Gamma \vdash e_1 : Bool$}
\AxiomC{$\Gamma \vdash e_2 : t_1$}
\AxiomC{$\Gamma \vdash e_3 : t_1$}
\TrinaryInfC{$\Gamma \vdash (if \ e_1 \ then \ e_2 \ else \ e_3) : t_1$}
\end{prooftree}
$$
In english: "**If** the environment proves that the condition expression of an if-then-else is of type $Bool$, and the environment proves that the expressions in both branches are of type $t_1$, **then** the environment proves that the expression $(if \ e_1 \ then \ e_2 \ else \ e_3)$ is of type $t_1$."

Interpretation: The condition expression in an if-then-else must be a boolean, and the expression in both conditional branches must match it in type. The entire construct evaluate to said type. 
### Generalization and instantiation, again
What I have described so far is not the full set of inference rules. We are missing the rule for instantiation and generalization. I find these easier to explain and understand informally, so I have omitted them. Refer to the previous section on the topic for more info.
# Unification and substitutions
When actually performing inference, unification is a process that often comes up. _Unification_ is a process that takes as input 2 types, and returns a substitution that when applied to either type, will make the 2 types equal. We use unification whenever we want to constrain 2 inferred types to be the same.

A _substitution_ is a list of bindings from names (of type variables) to concrete types. It can be _applied_ to any structure containing types, which will replace the type variables in said type with the type in their entry of the substitution, where applicable.

As an example, the types $a \xrightarrow{} bool \xrightarrow{} list(a)$ and $int \xrightarrow{} b \xrightarrow{} c$ can be unified using the substitution $\{a=int, b=bool, c=list(int)\}$, yielding the unified expression $int \xrightarrow{} bool \xrightarrow{} list(int)$. 

The typical algorithm for unifying 2 types is fairly simple, written below as F# code:

```sml
let rec unify (t1: Type) (t2: Type) : Map<string, Type> =
    match t1, t2 with
    | a, b when a = b -> Map.empty
    | TypeVar a, b when not (occurs a b) -> Map.ofList [(a, b)]
    | a, TypeVar b when not (occurs b a) -> Map.ofList [(b, a)]
    | TypeArrow (l1, r1), TypeArrow (l2, r2) ->
        let s1 = unify l1 l2
        let s2 = unify (apply s1 r1) (apply s2 r2)
        compose s1 s2
    | _ -> failwith "Could not unify types"
```

In English, this roughly means:
- If the 2 types are the same yields the empty substitution
- If either type is a type variable, and does not occur in the other type, return a substitution binding the type variable to the other type.
- If we are unifying 2 function (arrow) types, unify the first type in each function type, apply the resulting substitution to the second type in each function type, and then unify those. Combine the 2 resulting substitutions into 1 and return that.
- All other cases fail to unify. This indicates a type error in the program.

The so-called 'occurs check' is there to prevent pairs of types such as $a$ and $list(a)$ from unifying, since that would result in an infinite type.

The `apply` function takes a substitution and a type, and applies that substitution to a type.
# Let-polymorphism
In HM, generalization happens only in 1 place: When inferring let bindings. This is known as let-polymorphism or let-generalization. This lets the type system handle expression such as:

```sml
let id = fun(x) -> x in (x true, x 3)
```

Where the name `id` has been bound to a polymorphic value of type $\forall a. a \xrightarrow{} a$. If one attempts to generalize in other places during inference, it will certainly cause issues and incorrectly inferred types.

This means concretely that, while we are inferring the type of a let expression, when we are about to put a new binding into the environment before inferring the body of the let, we generalize the type of that binding, and put the generalized type into the environment instead. Remember that generalization is just universally quantifying free type variables.
# Typeclasses
_Disclaimer: I don't really know what I am talking about_

HM is hard to extend without introducing a bunch of soundness holes. It can be done, though.

HM (as with many type systems) is essentially just glorified propositional logic. $p \xrightarrow{} q$ reads both as "the type of a function from p to q" in type-land and as "p implies q" in logic-land. Both are valid interpretations.

Haskell-style typeclasses under this interpretation are predicates. The Haskell type `Num a => a -> a -> a` written as predicate logic is $\forall a. Num(a) \xrightarrow{} a \xrightarrow{} a \xrightarrow{} a$.

One can extend the type scheme, which was previously the most general way to describe a type at our disposal, to also include a list of predicates. Each predicate could be represented as a tuple of an identifier and a type, indicating that the type 'is a member of' the typeclass described by the identifier. If one accumulates these predicates during inference, just as is done for substitutions, one can solve these constraints in the end to see if the program typechecks. The problem basically becomes a typical predicate-logic problem, and one that languages such as Prolog interestingly seem like they were made to solve.

Much more about these ideas in the "Typing Haskell in Haskell" paper linked to below.
# Links
https://gist.github.com/pema99/dab60ee4248eef2cff5e74e76d672620

[https://github.com/pema99/bonk/blob/master/src/Inference.fs](https://github.com/pema99/bonk/blob/master/src/Inference.fs)

[https://course.ccs.neu.edu/cs4410sp19/lec_type-inference_notes.html](https://course.ccs.neu.edu/cs4410sp19/lec_type-inference_notes.html)

[http://dev.stephendiehl.com/fun/006_hindley_milner.html#inference-monad](http://dev.stephendiehl.com/fun/006_hindley_milner.html#inference-monad)

[https://jgbm.github.io/eecs662f17/Notes-on-HM.html](https://jgbm.github.io/eecs662f17/Notes-on-HM.html)

[https://gist.github.com/chrisdone/0075a16b32bfd4f62b7b](https://gist.github.com/chrisdone/0075a16b32bfd4f62b7b)

#type-theory #math #hindley-milner #type-inference