Pratt 解析, 又称 Precedence climbing (优先级爬升)

可以用于解析 前缀运算、中缀运算、后缀运算

Pratt 的基本思想是让左结合与右结合进行竞争, 也正因如此 Pratt 较难直接处理非结合的运算

解析中缀时, 在一个中缀运算符之后, 希望该运算符右侧表达式的优先级比该运算符高

而如果中缀运算右结合, 则希望该运算符右侧的表达式优先级不小于该运算符

```python
# 优先级, 是否左结合
INFIX_PREC = {
        '+': (2, 1),
        '-': (2, 1),
        '*': (3, 1),
        '/': (3, 1),
        '^': (6, 0),
}
PREFIX_PREC = {
        '(': 1,
        '-': 4,
}
SUFFIX_PREC = {
        '!': 5,
}
INIT_PREC = PREFIX_PREC['(']


def parse_prefix(toker: Tokenizer) -> Node:
    prec = PREFIX_PREC.get(toker.cur())
    if prec is None:
        return Node(toker.bump(toker.cur()))

    match toker.cur():
        case '(':
            toker.bump('(')
            node = parse_infix(toker, prec)
            toker.bump(')')
            return node
        case op:
            toker.bump(op)
            return Node(op, [parse_infix(toker, prec)])


def parse_suffix(node: Node, toker: Tokenizer, min_prec) -> Node:
    op = toker.cur()
    prec = SUFFIX_PREC.get(op)
    if prec is None or prec < min_prec:
        return node
    toker.bump(op)
    node = Node(op, [node])
    return parse_suffix(node, toker, min_prec)


def parse_infix(toker: Tokenizer, min_prec) -> Node:
    node = parse_prefix(toker)
    while True:
        op = toker.cur()
        prec, left_assoc = INFIX_PREC.get(op, (-100, 0))
        if prec < min_prec:
            break
        toker.bump(op)
        node = Node(op, [node, parse_infix(toker, prec+left_assoc)])
    return parse_suffix(node, toker, min_prec)


def test() -> None:
    toker = Tokenizer("-2^3^4!!")
    node = parse_infix(toker, INIT_PREC)
    assert toker.cur() == "", toker.src
    print(node)


test()
```

词法:


```python
import re

class Tokenizer:
    """
    >>> toker = Tokenizer("123 + 456")
    >>> toker.cur()
    '123'
    >>> toker.cur()
    '123'
    >>> toker.bump("123")
    >>> toker.cur()
    '+'
    """
    TOKEN = re.compile(r"\s*(\d+|\S)")

    def __init__(self, src):
        self.src = src

    def bump(self, expected):
        """跳过一个token"""
        assert self.src.startswith(expected), \
                f"expected `{expected}`, but found `{self.src[:len(expected)]}`"
        self.src = self.src[len(expected):]
        return expected

    def cur(self):
        """获取当前位于的token"""
        tok = self.TOKEN.match(self.src)
        return tok.group(1) if tok else ""


class Node:
    """
    >>> print(Node("123"))
    123
    >>> print(Node("-", ["123", "456"]))
    (- 123 456)
    """
    def __init__(self, data, args=None):
        self.data = data
        self.args = args

    def __str__(self):
        if self.args:
            return f"({self.data} {' '.join(map(str, self.args))})"
        return self.data
```
