# FASTTypeScript + MoTion Playbook (Pharo)

This document is dedicated for writing MoTion patterns to match a FASTTypeScript model. Some may want to find the if/else clause in a TypeScript file, others may need the switch case, others want specifc search... This is why you can benefit from MoTion to express the patterns you want, and use FASTTypeScript metamodel which allows representing the AST of TypeScript in Pharo. The combination of both tools, will allow you to apply any search. 

## 1) Requirements / Packages
- [FASTTypescript (metamodel + parser)](https://github.com/moosetechnology/FASTTypescript)
- [MoTion](https://github.com/alesshosry/Motion)

## 2) Parse TypeScript and get the Program node

```smalltalk
string := '// some TypeScript code'.
res := FASTTypeScriptParser new parse: string.
"res will be a FASTTypeScriptModel; we need to access the #entities inside👇🏻"

"Program entity: is the top entity of every TypeScript AST"
prog := (res entities detect: [ :ent | ent class = FASTTypeScriptProgram ]) first
"first because a collection will be returned containing only one entity of type FASTTypeScriptProgram which is the top entity".
```

## 3) Core MoTion traversal idioms

### 3.1) Recursive descent (find a node anywhere)
Use `#'children*'` from `FASTTypeScriptProgram` to search anywhere in the AST subtree.

```smalltalk
pattern := FASTTypeScriptProgram % {
  #'children*' <=> SomeNodeClass % { } as: #aNode
}.
results := pattern collectBindings: { #aNode } for: prog.
```

### 3.2) Immediate child constraint (avoid cross-product matches)
Use `#genericChildren` (no `*`) with list patterns when you need **direct** structural relationships.

```smalltalk
pattern := SomeNodeClass % {
  #genericChildren <=> { #'*_'. ChildNodeClass % { } as: #child. #'*_' }
}.
"this means you are looking for a SomeNodeClass that has direct children ChildNodeClass and possibly other children #'*_' "
```

## 4) Example patterns

### 4.1) Switch: exactly 2 cases + 1 default (via SwitchBody children)
```smalltalk
pattern := FASTTypeScriptProgram % {
  #'children*' <=> FASTTypeScriptSwitchBody % {
    #'children' <=> {
      FASTTypeScriptSwitchCase % { } as: #case1.
      FASTTypeScriptSwitchCase % { } as: #case2.
      FASTTypeScriptSwitchDefault % { } as: #default
    }
  } as: #switchStmt
}.
results := pattern collectBindings: { #switchStmt. #case1. #case2. #default } for: prog.
```

### 4.2) If/Else: direct else clause (avoids descendant cross-products)
```smalltalk
pattern := FASTTypeScriptProgram % {
  #'children*' <=> FASTTypeScriptIfStatement % {
    #genericChildren <=> {
      #'*_'.
      FASTTypeScriptElseClause % { } as: #elseClause.
      #'*_'
    }
  } as: #ifStmt
}.
results := pattern collectBindings: { #ifStmt. #elseClause } for: prog.
```

### 4.3) Try/catch: checking nested try/catch
```smalltalk
pattern := FASTTypeScriptProgram % {
  #'children*' <=> FASTTypeScriptTryStatement % {
    #'children*' <=> FASTTypeScriptTryStatement % { } as: #innerTry
  } as: #outerTry
}.

results := pattern collectBindings: { #outerTry. #innerTry } for: prog. 
```

### 4.4) Counting 3 methods in class
```smalltalk
string := 'class Example {
  methodWithInt() {
    let x: int = 10;
    console.log(x);
  }
  methodWithoutInt() {
    let name: string = "hello";
    console.log(name);
  }
}'.

res := FASTTypeScriptParser new parse: string.
prog := (res entities select: [ :ent | ent class = FASTTypeScriptProgram ]) first.
 
pattern :=  FASTTypeScriptProgram % {
	#'children*' <=> FASTTypeScriptClassBody % {
  		#'children' <=>  {  
			FASTTypeScriptMethodDefinition % {} as: #m1.
			FASTTypeScriptMethodDefinition % {} as: #m2.      
  		}.
	}. 
}.
```

### 4.5) Check if method is empty
```smalltalk
typescriptCode := '
function emptyFunction() {}

function nonEmptyFunction() {
    console.log("Hello, world!");
}

class Example {
    emptyMethod() {}
    nonEmptyMethod() {
        return 42;
    }
}
'.

parsedModel := FASTTypeScriptParser new parse: typescriptCode.
prog := (parsedModel entities select: [ :ent | ent class = FASTTypeScriptProgram ]) first.

pattern := FASTTypeScriptProgram % {
	 #'children*' <=> FASTTypeScriptMethodDefinition % {
    	#'body' <=> FASTTypeScriptStatementBlock % {
        #'children' <=> {}
		}
	}as: #emptyFunction.
}.

bindings := pattern collectBindings: { #emptyFunction } for: prog. 
```

### 4.6) Check if any function or method contain empty else and retrieve the owner
```smalltalk
str := 'function processValue(a: number) {
    if (a > 10) {
        console.log("big");
    } else { 
    }
}

class Calculator {
    compute(x: number) {
        if (x === 0) {
            return 0;
        } else {
            console.log("not zero");
        }
    }

    check(y: number) {
        if (y < 5) {
            console.log("small");
        } else { 
        }
    }
}'.

res := FASTTypeScriptParser new parse: str.
prog := (res entities select: [ :ent | ent class = FASTTypeScriptProgram ]) first.

emptyElsePattern :=
FASTTypeScriptElseClause % {        
    #children <=> FASTTypeScriptStatementBlock % { 
        #children <=> {}
    }     
} as: #emptyElse.


functionOwnerPattern :=
FASTTypeScriptProgram % {
    #'children*' <=> FASTTypeScriptFunctionDeclaration %% {
        #name <=> #'@ownerName'.
        #'body>children*' <=> emptyElsePattern
    }
}.

methodOwnerPattern :=
FASTTypeScriptProgram % {
    #'children*' <=> FASTTypeScriptMethodDefinition %% {
        #name <=> #'@ownerName'.
        #'body>children*' <=> emptyElsePattern
    }
}.

(functionOwnerPattern collectBindings: { #ownerName } for: prog).
(methodOwnerPattern collectBindings: { #ownerName } for: prog).
```

## 5) Debugging tips

### 5.1) Inspect children to learn structure
```smalltalk
node genericChildren collect: #class.
node genericChildren.
```

### 5.2) Why you got too many matches
- Using `#'genericChildren*'` matches *descendants*, producing cross-products when nested structures exist.
- Prefer `#genericChildren` when you mean “direct child”.

### 6) FASTTypeScript entities
In order to express a pattern effitiently, MoTion needs to know the class names representing the entities of TypeScript. This is why we are listing them here in accordance to there slots that can be used while expressing a MoTio pattern. Knowing that this is not the final list, many properties are still missing and need to be adapted. This will be done in the near future. Meanwhile the list is:
```
FASTTypeScriptAbstractMethodSignature
FASTTypeScriptAccessibilityModifier
FASTTypeScriptAddingTypeAnnotation
FASTTypeScriptAmbientDeclaration
FASTTypeScriptArguments, slots: newExpressionArgumentsOwner 
FASTTypeScriptArray, slots: assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTTypeScriptArrayPattern, slots: assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTTypeScriptArrayType, slots: assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTTypeScriptArrowFunction, slots: assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTTypeScriptAsserts
FASTTypeScriptAssertsAnnotation, slots: functionDeclarationReturnTypeOwner 
FASTTypeScriptAssignmentPattern
FASTTypeScriptCallSignature
FASTTypeScriptCatchClause, slots: tryStatementHandlerOwner 
FASTTypeScriptClass, slots: assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTTypeScriptClassBody, slots: classDeclarationBodyOwner 
FASTTypeScriptClassDeclaration, slots: programClassDeclarationOwner body name parentLoopStatement statementContainer tWithDeclarationsDeclarationsOwner declarations modifiers 
FASTTypeScriptClassHeritage
FASTTypeScriptClassStaticBlock
FASTTypeScriptComment
FASTTypeScriptCompilationUnit
FASTTypeScriptComputedPropertyName
FASTTypeScriptConditionalType, slots: assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTTypeScriptConstraint
FASTTypeScriptConstructSignature
FASTTypeScriptConstructorType, slots: type parameters 
FASTTypeScriptDecorator
FASTTypeScriptDefaultType
FASTTypeScriptERROR, slots: source element 
FASTTypeScriptElseClause
FASTTypeScriptEnumAssignment
FASTTypeScriptEnumBody, slots: enumDeclarationBodyOwner 
FASTTypeScriptEnumDeclaration, slots: body name programEnumDeclarationOwner parentLoopStatement statementContainer tWithDeclarationsDeclarationsOwner declarations modifiers 
FASTTypeScriptEscapeSequence
FASTTypeScriptExistentialType
FASTTypeScriptExportClause
FASTTypeScriptExportSpecifier, slots: name alias 
FASTTypeScriptExpression, slots: assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTTypeScriptExtendsClause, slots: type_arguments 
FASTTypeScriptExtendsTypeClause, slots: type 
FASTTypeScriptFinallyClause, slots: tryStatementFinalizerOwner body 
FASTTypeScriptFlowMaybeType, slots: assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTTypeScriptFormalParameters, slots: functionTypeParametersOwner constructorTypeParametersOwner methodDefinitionParametersOwner functionDeclarationParametersOwner 
FASTTypeScriptFunctionDeclaration, slots: body name parameters return_type 
FASTTypeScriptFunctionSignature
FASTTypeScriptFunctionType, slots: parameters assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTTypeScriptGeneratorFunction
FASTTypeScriptGeneratorFunctionDeclaration
FASTTypeScriptGenericType, slots: name type_arguments assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTTypeScriptHashBangLine
FASTTypeScriptHtmlComment
FASTTypeScriptIdentifier, slots: enumDeclarationNameOwner exportSpecifierNameOwner functionDeclarationNameOwner indexSignatureNameOwner nestedIdentifierObjectOwner optionalParameterPatternOwner assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner exportSpecifierAliasOwner methodDefinitionReturnTypeOwner 
FASTTypeScriptImplementsClause
FASTTypeScriptImport
FASTTypeScriptImportAlias
FASTTypeScriptImportAttribute
FASTTypeScriptImportClause
FASTTypeScriptImportRequireClause
FASTTypeScriptImportSpecifier
FASTTypeScriptIndexSignature, slots: index_type type name 
FASTTypeScriptIndexTypeQuery
FASTTypeScriptInferType
FASTTypeScriptInterfaceBody, slots: interfaceDeclarationBodyOwner 
FASTTypeScriptInterfaceDeclaration, slots: programInterfaceDeclarationOwner name body parentLoopStatement statementContainer tWithDeclarationsDeclarationsOwner declarations modifiers 
FASTTypeScriptInternalModule
FASTTypeScriptIntersectionType
FASTTypeScriptJsxText
FASTTypeScriptLexicalDeclaration
FASTTypeScriptLiteral, slots: primitiveValue assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTTypeScriptLiteralType
FASTTypeScriptLookupType
FASTTypeScriptMappedTypeClause
FASTTypeScriptMetaProperty
FASTTypeScriptMethodDefinition, slots: return_type name body parameters 
FASTTypeScriptMethodSignature
FASTTypeScriptModule
FASTTypeScriptNamedImports
FASTTypeScriptNamespaceExport
FASTTypeScriptNamespaceImport
FASTTypeScriptNestedIdentifier, slots: object property 
FASTTypeScriptNestedTypeIdentifier
FASTTypeScriptObjectAssignmentPattern
FASTTypeScriptObjectPattern
FASTTypeScriptObjectType, slots: assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTTypeScriptOmittingTypeAnnotation
FASTTypeScriptOptingTypeAnnotation
FASTTypeScriptOptionalChain
FASTTypeScriptOptionalParameter, slots: type pattern 
FASTTypeScriptOptionalType, slots: assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTTypeScriptOverrideModifier
FASTTypeScriptPair
FASTTypeScriptPairPattern
FASTTypeScriptParenthesizedType, slots: assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTTypeScriptPredefinedType, slots: assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTTypeScriptPrivatePropertyIdentifier, slots: assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTTypeScriptProgram, slots: classDeclarations enumDeclarations interfaceDeclarations source element 
FASTTypeScriptPropertyIdentifier, slots: nestedIdentifierPropertyOwner methodDefinitionNameOwner assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTTypeScriptPropertySignature
FASTTypeScriptPublicFieldDefinition
FASTTypeScriptReadonlyType, slots: assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTTypeScriptRegex, slots: assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTTypeScriptRegexFlags
FASTTypeScriptRegexPattern
FASTTypeScriptRequiredParameter
FASTTypeScriptRestPattern
FASTTypeScriptRestType, slots: assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTTypeScriptShorthandPropertyIdentifier
FASTTypeScriptShorthandPropertyIdentifierPattern
FASTTypeScriptSpreadElement
FASTTypeScriptStatement, slots: parentLoopStatement statementContainer 
FASTTypeScriptStringFragment
FASTTypeScriptSuper, slots: assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTTypeScriptSwitchBody
FASTTypeScriptSwitchCase
FASTTypeScriptSwitchDefault
FASTTypeScriptTemplateLiteralType, slots: assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTTypeScriptTemplateString
FASTTypeScriptTemplateSubstitution
FASTTypeScriptTemplateType, slots: assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTTypeScriptThis, slots: assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTTypeScriptThisType, slots: assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTTypeScriptTupleType, slots: assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTTypeScriptTypeAliasDeclaration
FASTTypeScriptTypeAnnotation, slots: indexSignatureTypeOwner optionalParameterTypeOwner assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner exportSpecifierAliasOwner methodDefinitionReturnTypeOwner functionDeclarationReturnTypeOwner 
FASTTypeScriptTypeArguments, slots: extendsClauseTypeArgumentsOwner genericTypeTypeArgumentsOwner newExpressionTypeArgumentsOwner assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner exportSpecifierAliasOwner methodDefinitionReturnTypeOwner 
FASTTypeScriptTypeAssertion, slots: assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTTypeScriptTypeIdentifier, slots: classDeclarationNameOwner constructorTypeTypeOwner extendsTypeClauseTypeOwner genericTypeNameOwner indexSignatureIndexTypeOwner interfaceDeclarationNameOwner assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTTypeScriptTypeParameter, slots: assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTTypeScriptTypeParameters, slots: assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTTypeScriptTypePredicate, slots: assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTTypeScriptTypeQuery, slots: assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTTypeScriptUndefined, slots: assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTTypeScriptUnionType, slots: assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTTypeScriptVariableDeclaration
FASTTypeScriptVariableDeclarator
FASTTypeScriptAbstractClassDeclaration
FASTTypeScriptAsExpression
FASTTypeScriptAssignmentExpression
FASTTypeScriptAugmentedAssignmentExpression
FASTTypeScriptAwaitExpression
FASTTypeScriptBinaryExpression
FASTTypeScriptCallExpression
FASTTypeScriptFunctionExpression
FASTTypeScriptInstantiationExpression
FASTTypeScriptMemberExpression
FASTTypeScriptNewExpression, slots: arguments type_arguments 
FASTTypeScriptNonNullExpression
FASTTypeScriptObject
FASTTypeScriptParenthesizedExpression, slots: withStatementObjectOwner ifStatementConditionOwner 
FASTTypeScriptSatisfiesExpression
FASTTypeScriptSequenceExpression
FASTTypeScriptSubscriptExpression
FASTTypeScriptTernaryExpression
FASTTypeScriptUnaryExpression
FASTTypeScriptUpdateExpression
FASTTypeScriptYieldExpression
FASTTypeScriptBoolean
FASTTypeScriptNull
FASTTypeScriptNumber
FASTTypeScriptString
FASTTypeScriptBreakStatement
FASTTypeScriptContinueStatement
FASTTypeScriptDebuggerStatement
FASTTypeScriptDoStatement
FASTTypeScriptEmptyStatement
FASTTypeScriptExportStatement
FASTTypeScriptExpressionStatement
FASTTypeScriptForInStatement, slots: body 
FASTTypeScriptForStatement
FASTTypeScriptIfStatement, slots: condition consequence 
FASTTypeScriptImportStatement
FASTTypeScriptLabeledStatement
FASTTypeScriptReturnStatement
FASTTypeScriptStatementBlock, slots: finallyClauseBodyOwner forInStatementBodyOwner functionDeclarationBodyOwner ifStatementConsequenceOwner methodDefinitionBodyOwner tryStatementBodyOwner withStatementBodyOwner 
FASTTypeScriptStatementIdentifier
FASTTypeScriptSwitchStatement
FASTTypeScriptThrowStatement
FASTTypeScriptTryStatement, slots: body finalizer handler 
FASTTypeScriptWhileStatement
FASTTypeScriptWithStatement, slots: body object 
FASTTypeScriptTypePredicateAnnotation
FASTTypeScriptFalse
FASTTypeScriptTrue
```
