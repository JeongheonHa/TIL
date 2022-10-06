# ✅ 중위 표기법을 후위 표기법으로 변환

대표적인 예제인 계산기의 연산과정을 구현해본다.

## ✅ main

```c
#include <stdio.h>
#include "InfixCalculator.h"

int main(void) {
    char exp1[] = "3-2+4";
    char exp2[] = "(1+2)*3";
    char exp3[] = "((1-2)+3)*(5-2)";
    char exp4[] = "((4+2)*4)-(3+2)*(4/2)";
    
    printf("%s = %d \n", exp1, evalInfixExp(exp1));
    printf("%s = %d \n", exp2, evalInfixExp(exp2));
    printf("%s = %d \n", exp3, evalInfixExp(exp3));
    printf("%s = %d \n", exp4, evalInfixExp(exp4));
    
    return 0;
}

/*
3-2+4 = 5 
(1+2)*3 = 9 
((1-2)+3)*(5-2) = 6 
((4+2)*4)-(3+2)*(4/2) = 14 
*/
```

## ✅ InfixToPostfix.h

```c
#ifndef InfixToPostfix_h
#define InfixToPostfix_h

void convertToProfixExp(char exp[]);

#endif /* InfixToPostfix_h */
```

## ✅ InfixToPostfix.c

```c
#include <stdio.h>
#include <stdlib.h>
#include <ctype.h>
#include <string.h>
#include "ListBaseStack.h"

int getOpPrec(char op) {
    switch(op) {
        case '*': case '/':
            return 5;
        case '+': case '-':
            return 3;
        case '(':
            return 1;
    }
    return -1;
}

int whoPrecOp(char op1, char op2) {
    int op1Prec = getOpPrec(op1);
    int op2Prec = getOpPrec(op2);
    
    if(op1Prec > op2Prec)
        return 1;
    else if (op1Prec < op2Prec)
        return -1;
    else
        return 0;
}

void convertToProfixExp(char exp[]) {
    Stack stack;    // 연산자를 위한 stack 선언
    int expLen = (int)strlen(exp);
    char* convExp = (char*)malloc(sizeof(char)*expLen+1);
    
    int i, idx = 0;
    char tok, popOp;
    
    memset(convExp, 0, sizeof(char)*expLen+1);
    stackInit(&stack);
    
    for(i = 0; i < expLen; i++) {
        tok = exp[i];
        if(isdigit(tok)) {  // 숫자라면
            convExp[idx++] = tok;
        } else {            // 연산자라면
            switch(tok) {
                case '(':
                    push(&stack, tok);
                    break;
                case ')':
                    while(1) {
                        popOp = pop(&stack);
                        if(popOp == '(')
                            break;
                        convExp[idx++] = popOp;
                    }
                    break;
                case '+': case '-':
                case '*': case '/':
                    while(!isEmpty(&stack) && whoPrecOp(peek(&stack), tok) >= 0)
                        convExp[idx++] = pop(&stack);
                    
                    push(&stack, tok);
                    break;
            }
        }
    }
    while(!isEmpty(&stack))     // 남아있는 연산자 스택에서 모두 꺼낸다.
        convExp[idx++] = pop(&stack);
    
    strcpy(exp, convExp);
    free(convExp);
}
```

## ✅ PostCalculator.h

```c
#ifndef PostCalculator_h
#define PostCalculator_h

int evalPostfixExp(char exp[]);

#endif /* PostCalculator_h */
```

## ✅ PostCalculator.c

```c
#include <ctype.h>
#include <string.h>
#include "ListBaseStack.h"

int evalPostfixExp(char exp[]) {
    Stack stack;
    stackInit(&stack);
    
    int expLen = (int)strlen(exp);
    char tok, op1, op2;
    
    for(int i = 0; i < expLen; i++) {
        tok = exp[i];
        
        if(isdigit(tok)) {
            push(&stack, tok-'0');
        } else {
            op2 = pop(&stack);
            op1 = pop(&stack);
            switch(tok) {
                case '+':
                    push(&stack, op1 + op2);
                    break;
                case '-':
                    push(&stack, op1 - op2);
                    break;
                case '*':
                    push(&stack, op1 * op2);
                    break;
                case '/':
                    push(&stack, op1 / op2);
                    break;
            }
        }
    }
    return pop(&stack);
}
```

## ✅ InfixCalculator.h

```c
#ifndef InfixCalculator_h
#define InfixCalculator_h

int evalInfixExp(char exp[]);

#endif /* InfixCalculator_h */
```

## ✅ InfixCalculator.c

```c
#include <stdlib.h>
#include <string.h>
#include "InfixCalculator.h"
#include "PostCalculator.h"
#include "InfixToPostfix.h"

int evalInfixExp(char exp[]) {
    int ret;
    int expLen = (int)strlen(exp);
    char* copyExp = (char*)malloc(sizeof(char)*expLen+1);
    strcpy(copyExp, exp);
    
    convertToProfixExp(copyExp);
    ret = evalPostfixExp(copyExp);
    
    free(copyExp);
    return ret;
}
```


