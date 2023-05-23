# ML in the bussines lascape 

## Machine learning (ML)

In short, A machine learning problem generally relates to a prediction when data is available. Machine Learning is the science (and art) of programming computers so they can learn from data [@Geron]. It is about extracting knowledge from data, a research field at the intersection of statistics, artificial intelligence, and computer science. It is also known as predictive analytics or statistical learning [@Muller].
Machine learning is about programming, but only some programming problems require ML. To detect if the problem we are facing is a ML problem, we need to have an objective, and what the benefit is for the company (client) we apply in business [@BF].


In this paragraph, we describe some examples. When describing those examples, we use some terminology that may sound unfamiliar to you, but we will cover it in further chapters. 

Understanding the goal allows us to determine what kind of data we expect to handle and the models we could apply. Suppose we are facing a problem in the housing market, and the business objective is to detect investment opportunities such as buying sub-valuated (in price) houses by predicting the housing prices. In this example, we would expect housing price data, such as housing location in latitude, longitude, median age z, total rooms, etc. In that case, we may apply supervised models such as linear regression and evaluate the model performance with RMSE. Another example in the financial sector could be predicting if a new bank customer would default on a loan or not (will repay the loan or not). In that case, it would be a classification problem, and we could use models such as logit or LDA and measure the performance with the Confusion Matrix.
On the other hand, in the same housing prices example, if we already have the information we mentioned in the last paragraph, and we want to know how crime affects the price in some areas. We could solve it by linear regression, but it wúltnd be a ML problem, but a causality one. It would be a ML problem if we want to predict the house prices in certain areas where crime has increased. Even when that would be an ML problem, it wouldn´t necessarily benefit our client or us. For example, if our client or we are a housing builder, it could help to decide where to build. Still, if the client is unaffected by the relationship between crime-house prices, then it would be an ML problem, but it is not benefiting our client or us.

In conclusion, before handling data and running algorithms, we suggest establishing a business goal, detecting if it is an ML problem, and if it would benefit the company (or client). 


## Variables terminology and notation

Many of the models we will cover in this book are of this kind:

$$y=\alpha_{0}\ +\beta_{1}\ x_{1}+\beta_{2}\ x_{2}+...+\beta_{n}\ x_{n}+e$$
Where $y$ is called the dependent variable, but also in other materials are, called the explained, output variable or response variable. On the other hand, $x$ are called independent variables and input variables, predictors or features. The $\beta´s$ are the parameters to be estimated, and $e$ is the error term. 

In regression, the idea is to estimate the parameters $\beta_{1}, \beta_{2},...,\beta_{n}$, to predict the value of $y$. When that happens, we represent the predicted values and estimated parameters as:


$$\hat{y}=\beta_{0}\ +\hat{\beta_{1}}\ x_{1}+\hat{\beta_{1}}\ x_{2}+...+\ \hat{\beta_{1}}\ x_{n}$$
Also, if we compare $y_{i}$  and a predicted value, we call that the residual, usually denoted by $\hat{e}$. It is defined as:

$$\hat{e_{i}}=y_{i}-\hat{y_{i}}=y_{i}-\beta_{0}\ -\hat{\beta_{1}}\ x_{1i}-\hat{\beta_{1}}\ x_{2i}-,...,-\ \hat{\beta_{1}}\ x_{ni}$$
In other words, each $\hat{e_{i}}$ is the residual for each observation. For example, if we have a data set with $n$ variables and $m$ observations, there would be $m$  residuals.
 

