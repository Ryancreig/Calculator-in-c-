#include <stdio.h>
#include <math.h>

// Structure to represent a mathematical expression
typedef struct Expression {
    char operation; // '+', '-', '*', '/', 's' (sin), 'c' (cos), 't' (tan), 'x' (variable)
    struct Expression* left;
    struct Expression* right;
} Expression;

// Function to calculate the derivative of an expression
Expression* derivative(Expression* expr) {
    Expression* result = malloc(sizeof(Expression));

    switch (expr->operation) {
        case 'x': // Variable
            result->operation = '1'; // Derivative of x is 1
            break;
        case 's': // Sin
            result->operation = 'c'; // Derivative of sin(x) is cos(x)
            result->left = derivative(expr->left);
            break;
        case 'c': // Cos
            result->operation = '-'; // Derivative of cos(x) is -sin(x)
            result->left = derivative(expr->left);
            break;
        case 't': // Tan
            result->operation = '1'; // Derivative of tan(x) is sec^2(x)
            result->left = malloc(sizeof(Expression));
            result->left->operation = '^';
            result->left->left = malloc(sizeof(Expression));
            result->left->left->operation = 's'; // sec
            result->left->left->left = expr->left;
            result->left->right = malloc(sizeof(Expression));
            result->left->right->operation = '2';
            break;
        case '+': // Addition
            result->operation = '+';
            result->left = derivative(expr->left);
            result->right = derivative(expr->right);
            break;
        case '-': // Subtraction
            result->operation = '-';
            result->left = derivative(expr->left);
            result->right = derivative(expr->right);
            break;
        case '*': // Multiplication
            result->operation = '+';
            result->left = malloc(sizeof(Expression));
            result->left->operation = '*';
            result->left->left = derivative(expr->left);
            result->left->right = expr->right;
            result->right = malloc(sizeof(Expression));
            result->right->operation = '*';
            result->right->left = expr->left;
            result->right->right = derivative(expr->right);
            break;
        case '/': // Division
            result->operation = '/';
            result->left = malloc(sizeof(Expression));
            Expression* numerator = malloc(sizeof(Expression));
            numerator->operation = '-';
            numerator->left = malloc(sizeof(Expression));
            numerator->left->operation = '*';
            numerator->left->left = derivative(expr->left);
            numerator->left->right = expr->right;
            numerator->right = malloc(sizeof(Expression));
            numerator->right->operation = '*';
            numerator->right->left = expr->left;
            numerator->right->right = derivative(expr->right);
            result->left = numerator;
            result->right = malloc(sizeof(Expression));
            result->right->operation = '^';
            result->right->left = expr->right;
            result->right->right = malloc(sizeof(Expression));
            result->right->right->operation = '2';
            break;
        default:
            printf("Invalid operation: %c\n", expr->operation);
            free(result);
            return NULL;
    }
    return result;
}

// Function to free the memory used by an expression
void freeExpression(Expression* expr) {
    if (expr == NULL)
        return;
    freeExpression(expr->left);
    freeExpression(expr->right);
    free(expr);
}

// Function to print an expression
void printExpression(Expression* expr) {
    if (expr == NULL)
        return;
    if (expr->operation == 'x') {
        printf("x");
    } else if (expr->operation == '1') {
        printf("1");
    } else {
        printf("(");
        printExpression(expr->left);
        printf(" %c ", expr->operation);
        printExpression(expr->right);
        printf(")");
    }
}

int main() {
    // Example: Calculate the derivative of sin(x) + x^2
    Expression* sinX = malloc(sizeof(Expression));
    sinX->operation = 's';
    sinX->left = malloc(sizeof(Expression));
    sinX->left->operation = 'x';

    Expression* xSquared = malloc(sizeof(Expression));
    xSquared->operation = '^';
    xSquared->left = malloc(sizeof(Expression));
    xSquared->left->operation = 'x';
    xSquared->right = malloc(sizeof(Expression));
    xSquared->right->operation = '2';

    Expression* sum = malloc(sizeof(Expression));
    sum->operation = '+';
    sum->left = sinX;
    sum->right = xSquared;

    Expression* result = derivative(sum);

    printf("The derivative of ");
    printExpression(sum);
    printf(" is ");
    printExpression(result);
    printf("\n");

    freeExpression(sum);
    freeExpression(result);

    return 0;
}

