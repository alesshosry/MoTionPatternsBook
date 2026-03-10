# FASTJava + MoTion Playbook (Pharo)

This document is dedicated for writing MoTion patterns to match a FASTJava model. Some may want to find the if/else clause in a Java file, others may need the switch case, others want specific search... This is why you can benefit from MoTion to express the patterns you want, and use FASTJava metamodel which allows representing the AST of Java in Pharo. The combination of both tools, will allow you to apply any search. 

## 1) Requirements / Packages
- [FASTJava (metamodel + parser)](https://github.com/moosetechnology/FAST-Java)
- [MoTion](https://github.com/alesshosry/Motion)

## 2) Parse Java and get the top node

Well, FAST-Java is not based on treesitter as other FAST metamodels (FASTXML, FASTTypeScript ...). So dealing with FAST-Java parsing will be a bit different. The main difference is that there is no specific top node common for all the ASTs that are generated. This will result a difficulty in knowing which node should be selected and what i sthe top type in MoTion pattern ready for the match.

There are two ways to parse Java code: one using #parseCodeMethodString: to parse Java methods and another one using #parseCodeString: to parse any encapsulated Java code (in methods, classes, interfaces ...). 

For example:

```smalltalk
parsedClass := JavaSmaCCProgramNodeImporterVisitor new parseCodeString: 'public class Account {
    private int customerNumber;
}'.

"parsedClass will be a FASTJavaModel containing list of entities, each one of them representing a node of the code; we need to access these #entities and select the top node; In this case it will be a FASTJavaClassDeclaration and we need the below to be able to retrieve it:" 

class := parsedClass allWithType: FASTJavaClassDeclaration. 
"from the model that was generated, we extract in the FASTJavaClassDeclaration which is the top node of all other entities; and from this entitiy we can start accessing other nodes by traversing the children"

"Another example to parse a method:"
parsedMethod := JavaSmaCCProgramNodeImporterVisitor new parseCodeMethodString: 'public void check() {
        System.out.println("Checking Account Balance"); 
}'.

"parsedMethod will be a FASTJavaModel; we need to access the #entities inside this model and retrieve FASTJavaMethodEntity which is the top entity that allow us to access the children nodes. To retrieve you can use: " 
method := parsedClass allWithType: FASTJavaMethodEntity.
``` 

Cases for imports in Java file followed by encapsulation (like class), if you want to match them, then you need to retrieve them and retrieve the class. If you need only the class content, no need to retrieve them.

## 3) Core MoTion traversal idioms

### 3.1) Recursive descent (find a node anywhere)
Use `#'children*'` from the top node to search anywhere in the AST subtree.

```smalltalk
pattern := FASTJavaClassDeclaration % {
  #'children*' <=> SomeNodeClass % { } as: #aNode
}.
results := pattern collectBindings: { #aNode } for: class.
```

### 3.2) Immediate child constraint (avoid cross-product matches)
Use `#children` (no `*`) with list patterns when you need **direct** structural relationships.

```smalltalk
pattern := SomeNodeClass % {
  #children <=> { #'*_'. ChildNodeClass % { } as: #child. #'*_' }
}.
"this means you are looking for a SomeNodeClass that has direct children ChildNodeClass and possibly other children #'*_' "
```

## 4) Example patterns

### 4.1) Switch: exactly 2 cases + 1 default (via SwitchBody children)
```smalltalk
pattern :=  
results := pattern collectBindings: { #switchStmt. #case1. #case2. #default } for: prog.
```

### 4.2) If/Else: direct else clause (avoids descendant cross-products)
```smalltalk
pattern :=  
results := pattern collectBindings: { #ifStmt. #elseClause } for: prog.
```

### 4.3) Try/catch: checking nested try/catch
```smalltalk
pattern := 
results := pattern collectBindings: { #outerTry. #innerTry } for: prog. 
```

### 4.4) Counting 3 methods in class
```smalltalk
string := 
```

### 4.5) Check if method is empty
```smalltalk
 
```

### 4.6) Check if any function or method contain empty else and retrieve the owner
```smalltalk
 
```

## 5) Debugging tips

### 5.1) Inspect children to learn structure
```smalltalk
node children collect: #class.
node children.
```

### 5.2) Why you got too many matches
- Using `#'children*'` matches *descendants*, producing cross-products when nested structures exist.
- Prefer `#children` when you mean “direct child”.

### 6) FASTJava entities
In order to express a pattern effitiently, MoTion needs to know the class names representing the entities of Java. This is why we are listing them here in accordance to there slots that can be used while expressing a MoTion pattern. In Java the list is more updated than any other metamodel because FASTJava has been implemented long time ago.
```
 FASTJavaAnnotation, slots: elements arrayOwner parentAnnotation javaModifierOwner name invokedIn 
FASTJavaArrayAccess, slots: javaVariableAssignmentOwner receiverOwner assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTJavaArrayAnnotationElement, slots: values arrayOwner parentAnnotation assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTJavaArrayInitializer, slots: arrayOwner parentAnnotation assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTJavaAssertStatement, slots: parentLoopStatement statementContainer 
FASTJavaAssignmentExpression, slots: operator arrayOwner parentAnnotation receiverOwner variable assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTJavaBreakStatement, slots: parentLoopStatement statementContainer 
FASTJavaCaseStatement, slots: javaCaseStatementSwitchOwner fastBehaviouralParent statements parentLoopStatement statementContainer 
FASTJavaCastExpression, slots: type receiverOwner assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTJavaCatchPartStatement, slots: tryOwner body catchedTypes parameter parentLoopStatement statementContainer 
FASTJavaClassDeclaration, slots: compilationUnit interfaces superclass javaDeclarationOwner declarations modifiers name invokedIn parentLoopStatement statementContainer source element 
FASTJavaComment, slots: content container 
FASTJavaCompilationUnit, slots: packageDeclaration importDeclarations classDeclarations interfaceDeclarations enumDeclarations 
FASTJavaConditionalExpression, slots: receiverOwner assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTJavaContinueStatement, slots: parentLoopStatement statementContainer 
FASTJavaDoWhileStatement, slots: parentLoopStatement statementContainer 
FASTJavaEmptyDimExpression, slots: assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTJavaEnumDeclaration, slots: compilationUnit constants interfaces javaDeclarationOwner declarations modifiers name invokedIn parentLoopStatement statementContainer source element 
FASTJavaExpressionStatement, slots: expression parentLoopStatement statementContainer 
FASTJavaFieldAccess, slots: fieldName javaVariableAssignmentOwner receiverOwner receiver assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTJavaForEachStatement, slots: fieldname type parentLoopStatement statementContainer 
FASTJavaForStatement, slots: parentLoopStatement statementContainer 
FASTJavaIfStatement, slots: parentLoopStatement statementContainer 
FASTJavaImportDeclaration, slots: isStatic isOnDemand compilationUnit qualifiedName 
FASTJavaInfixOperation, slots: operator arrayOwner parentAnnotation receiverOwner assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTJavaInitializer, slots: isStatic javaDeclarationOwner source element 
FASTJavaInterfaceDeclaration, slots: compilationUnit interfaces javaDeclarationOwner declarations modifiers name invokedIn parentLoopStatement statementContainer source element 
FASTJavaLabeledStatement, slots: label parentLoopStatement statementContainer 
FASTJavaLambdaExpression, slots: assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner parameters 
FASTJavaLiteral, slots: arrayOwner parentAnnotation receiverOwner 
FASTJavaMethodEntity, slots: type throws typeParameters javaDeclarationOwner modifiers statementBlock parameters name invokedIn source element 
FASTJavaMethodInvocation, slots: receiverOwner receiver invoked assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner arguments name invokedIn 
FASTJavaMethodReference, slots: receiver assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner name invokedIn 
FASTJavaModifier, slots: token javaModifierOwner assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTJavaNewExpression, slots: type receiverOwner receiver assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner arguments 
FASTJavaOuterThis, slots: type receiverOwner assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTJavaPackageDeclaration, slots: compilationUnit qualifiedName 
FASTJavaParameter, slots: hasVariableArguments type variable assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner parentAssignmentExpression parameterOwner name invokedIn 
FASTJavaQualifiedName, slots: namespace namespaceOwner qualifiedNameOwner javaVariableAssignmentOwner name invokedIn 
FASTJavaReturnStatement, slots: expression parentLoopStatement statementContainer 
FASTJavaStatement, slots: parentLoopStatement statementContainer 
FASTJavaStatementBlock, slots: javaTryCatchOwner javaTryFinallyOwner javaCatchPartStatementOwner fastBehaviouralParent statements parentLoopStatement statementContainer 
FASTJavaSwitchStatement, slots: cases parentLoopStatement statementContainer 
FASTJavaSynchronizedStatement, slots: parentLoopStatement statementContainer 
FASTJavaThrowStatement, slots: parentLoopStatement statementContainer 
FASTJavaTryCatchStatement, slots: try catches finally resources parentLoopStatement statementContainer 
FASTJavaTypeName, slots: javaTypeNameTypeExpressionOwner javaTypeNameQualify arrayOwner parentAnnotation assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner name invokedIn 
FASTJavaTypeParameterExpression, slots: javaMethodTypeParameterOwner types assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner name invokedIn 
FASTJavaTypePattern, slots: type variable assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTJavaUnaryExpression, slots: operator isPrefixedUnaryExpression assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTJavaVarDeclStatement, slots: declarators javaTryResourceOwner type javaDeclarationOwner modifiers parentLoopStatement statementContainer 
FASTJavaVariableDeclarator, slots: varDeclOwner variable 
FASTJavaVariableExpression, slots: javaCastExpressionTypeOwner javaCatchParameterOwnler javaClassPropertyOwner javaMethodReferenceOwner javaMethodTypeOwner javaNewExpressionOwner javaOuterThisOwner javaParameterTypeOwner javaParameterVariableOwner javaTypePatternVariableOwner javaVarDeclTypeOwner javaVariableDeclaratorOwner javaVariableAssignmentOwner receiverOwner assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner name invokedIn 
FASTJavaWhileStatement, slots: parentLoopStatement statementContainer 
FASTJavaDefaultCaseStatement
FASTJavaLabeledCaseStatement
FASTJavaBooleanLiteral, slots: primitiveValue assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTJavaCharacterLiteral, slots: primitiveValue assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTJavaDoubleLiteral, slots: primitiveValue assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTJavaFloatLiteral, slots: primitiveValue assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTJavaIntegerLiteral, slots: primitiveValue assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTJavaLongLiteral, slots: primitiveValue assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTJavaNullLiteral, slots: primitiveValue assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTJavaStringLiteral, slots: primitiveValue assignedIn parentExpressionLeft parentExpressionRight parentConditionalStatement expressionStatementOwner returnOwner parentExpression argumentOwner 
FASTJavaEmptyMethodDeclaration
FASTJavaNewArray
FASTJavaNewClassExpression, slots: declarations 
FASTJavaQualifiedTypeName, slots: namespace 
FASTJavaClassProperty, slots: fieldName type arrayOwner parentAnnotation 
FASTJavaEnumConstant, slots: parentEnum 
FASTJavaIdentifier, slots: javaForEachStatementFieldNameOwner 
FASTJavaThis
FASTJavaTypeExpression, slots: catchOwner javaClassDeclarationInterfaceOwner javaClassDeclarationSuperclassOwner javaEnumDeclarationInterfaceOwner javaForEachStatementTypeOwner javaInterfaceDeclarationInterfaceOwner javaTypePatternTypeOwner 
FASTJavaArrayTypeExpression
FASTJavaClassTypeExpression, slots: javaMethodThrowsOwner typeParameterOwner typeName arguments 
FASTJavaPrimitiveTypeExpression
FASTJavaBooleanTypeExpression
FASTJavaByteTypeExpression
FASTJavaCharTypeExpression
FASTJavaDoubleTypeExpression
FASTJavaFloatTypeExpression
FASTJavaIntTypeExpression
FASTJavaLongTypeExpression
FASTJavaShortTypeExpression
FASTJavaVoidTypeExpression
```
