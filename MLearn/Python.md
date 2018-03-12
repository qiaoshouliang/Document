#Python学习





## Python 模块命令

- 安装模块 sudo pip3 install numpy 
- 卸载模块 sudo pip3 uninstall numpy
- 升级模块 sudo pip3 install -U numpy

## 正则表达式

![](https://morvanzhou.github.io/static/results/basic/13-10-01.png)



## 第一个Tensorflow



```python
import tensorflow as tf
import numpy as np

#
x_data = np.random.rand(100).astype(np.float32)
y_data = x_data * 21 + 0.3

Weights = tf.Variable(tf.random_uniform([1], -1.0, 1.0))
biases = tf.Variable(tf.zeros([1]))

y = Weights * x_data + biases
loss = tf.reduce_mean(tf.square(y - y_data))

print(loss)

optimizer = tf.train.GradientDescentOptimizer(0.6)
train = optimizer.minimize(loss)

init = tf.global_variables_initializer()

sess = tf.Session()
sess.run(init)

for step in range(21):
    sess.run(train)
    if step % 20 == 0:
        print(step, sess.run(Weights), sess.run(biases))
```