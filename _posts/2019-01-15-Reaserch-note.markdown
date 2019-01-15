# Everyday research note

#### 2019-01-15

* implemented triplet loss function
```
def triplet_loss(x):
    anchor, positive, negative = x

    positive_distance = tf.reduce_sum(tf.square(tf.subtract(anchor, positive)), 1)
    negative_distance = tf.reduce_sum(tf.square(tf.subtract(anchor, negative)), 1)

    single_loss = tf.add(tf.subtract(positive_distance, negative_distance), ALPHA)
    loss = tf.reduce_mean(tf.maximum(single_loss, 0.0), 0)

    return loss
```
Source - [Triplet Loss in Keras](https://codepad.co/snippet/F1uVDD5N)
