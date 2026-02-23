# FASTTypeScript + MoTion Playbook (Pharo)

This document is a living cookbook for writing MoTion patterns over the FASTTypeScript AST/model.

## 1) Requirements / Packages
- [FASTTypescript (metamodel + parser)](https://github.com/moosetechnology/FASTTypescript)
- [MoTion](https://github.com/alesshosry/Motion)

## 2) Parse TypeScript and get the Program node

```smalltalk
string := '// some TypeScript code'.
res := FASTTypeScriptParser new parse: string.
"res will be a FASTTypeScriptModel; we need to access the #entities inside👇🏻"

"Program entity: is the top entity of every TypeScript AST"
prog := (res entities detect: [ :ent | ent class = FASTTypeScriptProgram ]) first "first because a collection will be returned containing only one entity of type FASTTypeScriptProgram which is the top entity".
```

## 3) Core MoTion traversal idioms

### 3.1) Recursive descent (find a node anywhere)
Use `#'children*'` from `FASTTypeScriptProgram` to search anywhere in the AST subtree.

```smalltalk
pattern := FASTTypeScriptProgram % {
  #'children*' <=> (SomeNodeClass % { } as: #aNode)
}.
results := pattern collectBindings: { #aNode } for: prog.
```

### 3.2) Immediate child constraint (avoid cross-product matches)
Use `#genericChildren` (no `*`) with list patterns when you need **direct** structural relationships.

```smalltalk
pattern := SomeNodeClass % {
  #genericChildren <=> { #'*_'. (ChildNodeClass % { } as: #child). #'*_' }
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
  #'children*' <=> (FASTTypeScriptIfStatement % {
    #genericChildren <=> {
      #'*_'.
      (FASTTypeScriptElseClause % { } as: #elseClause).
      #'*_'
    }
  } as: #ifStmt)
}.
results := pattern collectBindings: { #ifStmt. #elseClause } for: prog.
```

### 4.3) Try/catch: checking nested try/catch
```smalltalk
pattern := FASTTypeScriptProgram % {
  #'children*' <=> (FASTTypeScriptTryStatement % {
    #'children*' <=> (FASTTypeScriptTryStatement % { } as: #innerTry)
  } as: #outerTry)
}.

results := pattern collectBindings: { #outerTry. #innerTry } for: prog. 
```

## 4) Debugging tips

### 4.1) Inspect children to learn structure
```smalltalk
node genericChildren collect: #class.
node genericChildren.
```

### 4.2) Why you got too many matches
- Using `#'genericChildren*'` matches *descendants*, producing cross-products when nested structures exist.
- Prefer `#genericChildren` when you mean “direct child”.
