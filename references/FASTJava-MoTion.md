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
"parsedClass will be a FASTJavaModel containing list of entities, each one of them representing a node of the code;" 

"Another example to parse a method:"
parsedMethod := JavaSmaCCProgramNodeImporterVisitor new parseCodeMethodString: 'public void check() {
        System.out.println("Checking Account Balance"); 
}'.

"parsedMethod will be a FASTJavaModel containing list of entities, each one of them representing a node of the code;"
``` 

Since the result of parsing is a FASTJavaModel, and since FAST-Java does not contain a top entity (like document for Typescript AST), then we recommend that any MoTion pattern to be matched is a FASTJavaModel pattern, and the entities will be accessed using #allModelEntities.
So mainly patterns for FASTJava will take this shape:

```smalltalk
pattern := FASTJavaModel % {    
		#'allModelEntities' <=> FASTJavaTheEntityIamLookingFor % { 
			....
		} as: #myEntity.
	}.
```

## 3) Example patterns

### 3.1) Switch: exactly 2 cases + 1 default (via SwitchBody children)
```smalltalk
string := 'public String getDayType(int day) {
    switch (day) {
        case 1:
            return "Weekday";
        case 2:
            return "Weekend";
        default:
            return "Invalid day";
    }
}'.


pattern := FASTJavaModel % {    
		#'allModelEntities' <=> FASTJavaSwitchStatement % { 
			#cases  <=> { 
				FASTJavaLabeledCaseStatement % {} as:#case1. 
				FASTJavaLabeledCaseStatement % {} as:#case2.  
				FASTJavaDefaultCaseStatement % {} as:#default. }
		} as: #amethod.
	}.

parsedCode := JavaSmaCCProgramNodeImporterVisitor new parseCodeMethodString: string.
  
results := pattern collectBindings: {  #amethod. #case1. #case2. #default } for: parsedCode. 
```

### 3.2) If/Else: get the else that is not followed by if
```smalltalk
string := 'public class NumberUtils
{
    public string ClassifyNumber(int number)
    {
        if (number < 0)
        {
            return "Negative number";
        }
        else if (number == 0)
        {
            return "Zero";
        }
        else if (number > 0 && number <= 10)
        {
            return "Small positive number";
        }
        else
        {
            return "Large positive number";
        }
    }

    public string GradeResult(int score)
    {
        if (score < 50)
        {
            return "Fail";
        }
        else if (score < 65)
        {
            return "Pass";
        }
        else if (score < 80)
        {
            return "Good";
        }
        else if (score <= 100)
        {
            return "Excellent";
        }
        else
        {
            return "Invalid score";
        }
    }
}'.

parsedCode := JavaSmaCCProgramNodeImporterVisitor new parseCodeString: string.

pattern := FASTJavaModel % {    
		#'allModelEntities' <=> FASTJavaIfStatement % { 
			#'elsePart'  <~=> FASTJavaIfStatement % { } .
		} as: #amethod
	}.

results := pattern collectBindings: { #amethod.} for: parsedCode.
```


## 4) Debugging tips

### 4.1) Inspect children to learn structure
```smalltalk
node children collect: #class.
node children.
```

### 4.2) Why you got too many matches
- Using `#'children*'` matches *descendants*, producing cross-products when nested structures exist.
- Prefer `#children` when you mean “direct child”.

## 5) FASTJava entities
In order to express a pattern efficiently, MoTion needs to know the class names representing the entities of Java. This is why we are listing them here in accordance to their slots that can be used while expressing a MoTion pattern. In Java the list is more updated than any other metamodel because FASTJava has been implemented long time ago.
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
