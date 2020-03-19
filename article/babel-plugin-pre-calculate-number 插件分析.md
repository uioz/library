# babel-plugin-pre-calculate-number 插件分析

```javascript
/**
 * see https://github.com/axetroy/babel-plugin-pre-calculate-number/blob/master/index.js
 * 
 * 这个 babel 插件执行逆向递归操作
 * 1. 符合递归条件
 *   1.1 执行操作
 *   1.2 调用父级执行执行操作 (1)
 * 2. 不符合条件略过
 * 
 * 这种逻辑得以执行依赖于, 访问者 设计模式和节点遍历的方式例如:
 * 1 + 2 + 3 = BinaryExpression(1) = BinaryExpression(2) + Literal = Literal + Literal + Literal
 * 迭代器会首先迭代 BinaryExpression(1) 但是它不符合执行的条件, 所以会执行 BinaryExpression(2)
 * BinaryExpression(2) 完成 Literal + Literal 的操作, 执行反向递归操作, 发现父级符合递归条件
 * 执行父级 BinaryExpression(1) 此时的父级已经成为 Literal + Literal 完成操作后并未继续发现符合递归的条件, 停止递归.
 */

module.exports = function({ types: t }) {
  function calcExpression(left, operator, right) {
    let result;
    switch (operator) {
      case "+":
        result = left + right;
        break;
      case "-":
        result = left - right;
        break;
      case "*":
        result = left * right;
        break;
      case "/":
        result = left / right;
        break;
      case "%":
        result = left % right;
        break;
      case ">>":
        result = left >> right;
        break;
      case ">>>":
        result = left >>> right;
        break;
      case "<<":
        result = left << right;
        break;
      case "|":
        result = left | right;
        break;
      case "&":
        result = left & right;
        break;
      case "^":
        result = left ^ right;
        break;
      case "**":
        result = left ** right;
        break;
      default:
    }
    return result;
  }

  const visitor = {
    BinaryExpression(path) {
      debugger;
      const node = path.node;
      let result;
      // example: Math.PI * (2 ** 2);
      if (
        (t.isMemberExpression(node.left) && t.isNumericLiteral(node.right)) ||
        (t.isNumericLiteral(node.left) && t.isMemberExpression(node.right))
      ) {
        const member = t.isMemberExpression(node.left) ? node.left : node.right;
        const number = t.isNumericLiteral(node.left) ? node.left : node.right;

        if (member.object && member.object.name === "Math") {
          let method;

          if (t.isIdentifier(member.property)) {
            method = member.property.name;
          } else if (t.isStringLiteral(member.property)) {
            method = member.property.value;
          }

          switch (method) {
            case "E":
            case "LN2":
            case "LN10":
            case "LOG2E":
            case "LOG10E":
            case "SQRT1_2":
            case "SQRT2":
            case "PI":
              result = calcExpression(
                node.left === member ? Math[method] : number.value,
                node.operator,
                node.right === number ? number.value : Math[method]
              );
              break;
            default:
          }
        }
      } else if (
        t.isNumericLiteral(node.left) &&
        t.isNumericLiteral(node.right)
      ) {
        result = calcExpression(
          node.left.value,
          node.operator,
          node.right.value
        );
      }

      // if got result, should update parent node
      if (result !== undefined) {
        path.replaceWith(t.numericLiteral(result));

        let parentPath = path.parentPath;

        // traverse parent node
        parentPath &&
          t.isBinaryExpression(parentPath.node) &&
          visitor.BinaryExpression.call(this, parentPath);
        parentPath &&
          t.isUnaryExpression(parentPath.node) &&
          visitor.UnaryExpression.call(this, parentPath);
      }
    },
    CallExpression(path) {
      const node = path.node;
      // only allow calc Math's method
      if (
        t.isMemberExpression(node.callee) &&
        t.isIdentifier(node.callee.object) &&
        node.callee.object.name === "Math"
      ) {
        let method;
        if (t.isIdentifier(node.callee.property)) {
          method = node.callee.property.name;
        } else if (t.isStringLiteral(node.callee.property)) {
          method = node.callee.property.value;
        }
        let result;

        // The argument of this callExpression
        const args = node.arguments;

        switch (method) {
          default:
            // If the all argument is numberic literal
            const isAllNumber = args.every(v => t.isNumericLiteral(v));
            if (isAllNumber) {
              result = Math[method].apply(
                Math,
                args.map(v => v.value)
              );
            }
        }

        if (result !== undefined) {
          path.replaceWith(t.numericLiteral(result));
        }
      }
    },
    UnaryExpression(path) {
      const node = path.node;
      if (t.isUnaryExpression(node) && t.isNumericLiteral(node.argument)) {
        let result;
        switch (node.operator) {
          case "+":
            result = +node.argument.value;
            break;
          case "-":
            result = -node.argument.value;
            break;
          case "~":
            result = ~node.argument.value;
            break;
          default:
        }

        if (result !== undefined) {
          path.replaceWith(t.numericLiteral(result));
          let parentPath = path.parentPath;
          if (parentPath) {
            if (t.isBinaryExpression(parentPath.node)) {
              // example: 5 + (~3);
              visitor.BinaryExpression.call(this, parentPath);
            }
            if (t.isUnaryExpression(parentPath.node)) {
              // example: -(~2);
              visitor.UnaryExpression.call(this, parentPath);
            }
          }
        }
      }
    }
  };

  return {
    visitor
  };
};

```

