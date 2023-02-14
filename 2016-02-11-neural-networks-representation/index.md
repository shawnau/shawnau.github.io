# Machine Learning Notes(8)-Neural Networks Representation


<!--more-->

## Neural Networks Model
1. A single neuron model: logistic unit <p align="center"> <img src="https://my-imgshare.oss-cn-shenzhen.aliyuncs.com/NN0.png" /> </p>
   - Takes 3+1 inputs(the extra input called bias is just like $\theta_0$ in logistic regression, not shown in picture).
   - Both input and output could be represented as vectors, in which each unit has its own parameters $\theta$
   - All the units in the same layer take the same input $\mathbf{x}$, as the pic shows.
   - Each unit has only one output: $sigmoid(\theta^T x)$. Of course there're other choices for sigmoid function.
2. Neural Networks <p align="center"> <img src="https://my-imgshare.oss-cn-shenzhen.aliyuncs.com/NN2_5.png" /> </p>
   - A neural network consists of an input Layer, an output layer and hidden layers.
   - In the model above, layer 1 is input layer, 2 is hidden layer and 3 is the output layer.

<!--more-->

## Calculation from one layer to the next
<p align="center"> <img src="https://my-imgshare.oss-cn-shenzhen.aliyuncs.com/NN2.png" /> </p>

In the picture above, we have the networks from layer j to layer j+1, in which layer j has 3(+1) units while layer j+1 has 3 layers. Let $s_j = 3$, $s_{j+1} = 3$
   - $\alpha^{(j)}$ : Output of the $j_{th}$ layer. $s_j + 1$ dimension vector.
   - $\theta_i^{(j)}$ : Parameters in the $i_{th}$ unit of ${(j+1)}_{th}$ layer. $s_j + 1$ dimension vector.
   - $\mathbf{\theta^{(j)}} = \begin{bmatrix} \theta_1^{(j)} & \theta_2^{(j)} & \cdots & \theta_{s_(j + 1)}^{(j)} \end{bmatrix}^T$ : All the network parameters from $j_{th}$ layer to ${(j+1)}_{th}$ layer.
   - We have: $\alpha^{(j+1)} = sigmoid(\mathbf{\theta^{(j)}}\alpha^{(j)})$ add $\alpha_0^{(j+1)}$


