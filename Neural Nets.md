$Z_{1}=W_{1}X+b_{1}$
$A_{1}=ReLU(Z_{1})$
$Z_{2}=W_{2}A_{1}+b_{2}$
$A_{2}=ReLU(Z_{2})$
$Z_{3}=W_{3}A_{2}+b_{3}$
$A_{3}=Softmax(Z_{3})$
$L(y, A_{3})=-\sum_{i}y_{i}\log(A_{3,i})$

$$\text{ReLU}'(x) = \begin{cases}
    1 & \text{if } x > 0 \\
    0  & \text{if } x \leq 0
\end{cases}
$$

## Layer 3
$$\delta_{3}=\frac{\partial L}{\partial Z_{3}}=A_{3}-y$$
$$\frac{\partial L}{\partial W_{3}}=\delta_{3}\frac{\partial Z_{3}}{\partial W_{3}}=(A_{3}-y)(A_{2})$$
$$\frac{\partial L}{\partial b_{3}}=\delta_{3}\frac{\partial Z_{3}}{\partial b_{3}}=(A_{3}-y)(1)$$

## Layer 2
$$\delta_{2}=\frac{\partial L}{\partial Z_{2}}=\delta_{3}\frac{\partial Z_{3}}{\partial A_{2}} \frac{\partial A_{2}}{\partial Z_{2}}=(A_{3}-y)(W_{3})(\text{ReLU'}(Z_{2}))$$
$$\frac{\partial L}{\partial W_{2}}=\delta_{2} \frac{\partial Z_{2}}{\partial W_{2}}=\delta_{2}A_{1}$$
$$\frac{\partial L}{\partial b_{2}}=\delta_{2} \frac{\partial Z_{2}}{\partial b_{2}}=\delta_{2}$$
## Layer 1
$$\delta_{1}=\frac{\partial L}{\partial Z_{1}}=\delta_{2} \frac{\partial Z_{2}}{\partial A_{1}} \frac{\partial A_{1}}{\partial Z_{1}}=\delta_{2}W_2\text{ReLU}'(Z_{1})$$
$$\frac{\partial L}{\partial W_{1}}=\delta_{1} \frac{\partial Z_{1}}{\partial W_{1}}=\delta_{1}X$$
$$\frac{\partial L}{\partial b_{1}}=\delta_{1} \frac{\partial Z_{1}}{\partial b_{1}}=\delta_{1}$$

# Shapes
```Python
I = 28 ** 2
n = 64
O = 10
B = 32 # batch size
Z1 = X @ W1 + B1 
X.shape = (B, I)
W1.shape = (I, n) # 64 hidden
B1.shape = (1, n)
A1.shape = Z1.shape = (B, n)

Z2 = A1 @ W2 + B2
W2.shape = (n, n)
B2.shape = (1, n)
A2.shape = Z2.shape = (B, n)

Z3 = A2 @ W3 + B3
W3.shape = (n, O)
B3.shape = (1, O)
A3.shape = Z3.shape = (B, O)
```

```python
A3.shape = (32, 10)
y.shape = (32, 10)
delta3 = A3 - y
delta3.shape = (32, 10)

# we know from earlier that dW3 must be delta3 * A2
# we know that dW3.shape = W3.shape = (64, 10)
A2.shape = (32, 64)
# A2.T forces 64 rows, delta 3 forces 10 columns, therefore:
dW3 = A2.T @ delta3

# we know that db3 = delta3
# we know that db3.shape = b3.shape = (1, 10)
# we must collapse delta3.shape (which is (32, 10)) into (1, 10)
# we can sum up all the contributions from each sample, and collapse them
db3 = delta3.sum(axis=0, keepdims=true)
# HOWEVER, it's more common to consider an average here:
db3 = delta3.sum(axis=0, keepdims=true) / B

W3.shape = (64, 10)
delta3.shape = (32, 10)
delta2 = delta3 @ W3.T * ReLU'(Z3)
delta2.shape = (64, 32)
```

generically, we find that for back prop:
$dW_{l}=A_{l-1}^T\delta_{l}$ and $db_{l} = \frac{1}{B}\sum_{i=1}^B\delta_{l}^i$


## Put a little bit cleaner...


We model our network as the following:

A feed forward layer:
$Z_{l}=A_{l-1}W_{l}+b_{l}$
An activation function:
$A_{l}=\text{ReLU}(Z_{l})$ where 
$$\text{ReLU}(x) = \begin{cases}
    x & \text{if } x > 0 \\
    0  & \text{if } x \leq 0
\end{cases}
$$

And an output function that normalizes to 1, namely, a softmax:
$$Softmax(z)_{i}=\frac{e^{zi}}{\sum_{j}e^{zj}}$$
A forward pass is as simple as passing inputs in and passing outputs layer to layer.
We measure loss in this case with categorical cross entropy loss, defined as:
$$L(y, \hat{y})=-\sum_{i}y_{i}\log(\hat{y}_{i})$$
backprop begins with the derivative of our loss function. We first define an error term:
$\delta_{l}=\frac{\partial L}{\partial Z_{l}}=(\delta_{l+1}W_{l+1}) \odot \text{ReLU}(Z_{l})$

in the case of the final output layer, specifically with our loss function and softmax, we obtain:
$\delta=A_{n}-y=\hat{y}-y$, where $y$ is a one hot encoding of our label vector.

back prop is as simple as $W_{l}-dW_{l}*S$ and $b_{l}-db_{l}*S$ where $S$ is a step value, where
$dW_l=\frac{1}{B}A_{{l-1}}^T\delta_{l}$
$db_{l}=\frac{1}{B}\sum_{i=1}^B\delta_{l}^i$