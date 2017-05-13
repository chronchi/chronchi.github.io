---
layout: post
title:  Método do gradiente e gradiente estocástico
categories:
  - julia
  - machine learning
description: Implementação dos métodos do gradiente e gradiente estocástico.
permalink: /blog/:title.html
---


Cost functions and prediction functions
======


```julia
#LMS cost function

function cost_function(w,x,y)
  m = length(y)
  n = length(w)
  a = zeros(m,1)
  for i = 1:m
    a[i] = (dot(w,x[i,:]) - y[i])^2
  end
  a = sum(a)/(2*m)
  return a
end
```




```julia
function cost_function_classification(w,x,y)
    m = length(y)
    n = length(w)
    a = zeros(m,1)
    for i = 1:m
        a[i] = y[i]*log(predict(w,x[i,:])) + (1-y[i])*log(1-predict(w,x[i,:]))
    end
    a = sum(a)
    return a
end
```






    cost_function_classification (generic function with 1 method)




```julia
function sigmoid(x::Float64)
    #sigmoid function
    y = exp(x)/(1+exp(x))
    return y
end
```




    sigmoid (generic function with 1 method)




```julia
function identityy(x::Float64)
    # identity function
    return x
end
```




    identityy (generic function with 1 method)




```julia
predict(w,x) = sigmoid(dot(w,x))
```




    predict (generic function with 1 method)




```julia
function cost_function_classification(w,x,y)
    m = length(y)
    n = length(w)
    a = zeros(m,1)
    for i = 1:m
        a[i] = y[i]*log(predict(w,x[i,:])) + (1-y[i])*log(1-predict(w,x[i,:]))
    end
    a = sum(a)
    return a
end
```




    cost_function_classification (generic function with 1 method)



Gradient descent/Ridge Regression
===


```julia
function grad_cost_function(w,x,y)
  m = length(y)
  n = length(w)
  a = zeros(n)
  for j = 1:n
    for i = 1:m
       a[j] += (dot(w,x[i,:]) - y[i])*x[i,j]
    end
  end
  return a
end
```


```julia
function trainer(w,x,y;cost = cost_function ,grad = grad_cost_function,
  max_iter = 10000, tol = 1e-4, α = 1)

  m = length(y)

  f(w) = cost(w,x,y)
  g(w) = grad(w,x,y)

  iter = 0
  fw = f(w)
  gw = g(w)
  start_time = time()
  elapsed_time = 0.0
  while norm(gw) > tol && iter < max_iter
    #while cost_function(w + α*gw,x,y) > cost_function(w,x,y) + α*dot(gw,gw)
    #  α = 0.9 * α
    #end
    w = w - (α/m) *gw
    fw = f(w)
    gw = g(w)
    iter += 1
    elapsed_time = time() - start_time
  end
  return w, iter, elapsed_time
end
```



Gradient descent/Logistic Regression
===


```julia
function gradclass(w,x,y)
    m = length(y)
    n = length(w)
    a = zeros(n)
    for j = 1:n
        for i = 1:m
            a[j] += (y[i]-predict(w,x[i,:]))*x[i,j]
        end
    end
    return a
end
```




    gradclass (generic function with 1 method)




```julia
function trainerclass(w,x,y;cost = cost_function_classification ,grad = gradclass,
  max_iter = 10000, tol = 1e-4, α = 1)

  m = length(y)

  f(w) = cost(w,x,y)
  g(w) = grad(w,x,y)

  iter = 0
  fw = f(w)
  gw = g(w)
  start_time = time()
  elapsed_time = 0.0
  while norm(gw) > tol && iter < max_iter
    #while cost_function(w + α*gw,x,y) > cost_function(w,x,y) + α*dot(gw,gw)
    #  α = 0.9 * α
    #end
    w = w - (α/m) *gw
    fw = f(w)
    gw = g(w)
    iter += 1
    elapsed_time = time() - start_time
  end
  return w, iter, elapsed_time
end
```




    trainerclass (generic function with 1 method)



Stochastic Gradient Descent/Ridge Regression
=======


```julia
function gradsgd(w,x,y,i)
    m = length(y)
    n = length(w)
    a = (dot(w,x[i,:]) - y[i]) * x[i,:]
    return a
end
```




```julia
function sgdm(w_init,x,y; max_iter = 10000, tol =
    1e-3, α = 1, max_time = 30.0)

    m = length(y)
    w = copy(w_init)
    w_new = zeros(length(w))

    g(w,i) = gradsgd(w,x,y,i)

    iter = 0
    start_time = time()
    elapsed_time = 0.0

    idx = collect(1:m)
    shuffle!(idx)
    i = 1

    gw = ones(length(w))

    while norm(gw) > tol && iter < max_iter
        i = mod(i+1,m)
        #γ = α/(1 + α*iter)
        gw = g(w,idx[i])
        w_new = w - α * gw
        w = w_new
        gw = g(w,idx[i])
        iter += 1
        elapsed_time = time() - start_time
        if elapsed_time > max_time
          break
        end
    end
    return w, iter, elapsed_time
end
```




    sgdm (generic function with 1 method)



Stochastic Gradient Descent/Logistic Regression with cross entropy loss
=======


```julia
function gradsgdclass(w,x,y,i)
    a = (y[i] - predict(w,x[i,:])) * x[i,:]
    return a
end
```



```julia
function sgdclass(w_init::Vector,x,y; max_iter = 10000, tol =
    1e-3, α = 1, max_time = 30.0)

    m = length(y)
    w = copy(w_init)
    w_new = zeros(length(w))

    g(w,i) = gradsgd(w,x,y,i)

    iter = 0
    start_time = time()
    elapsed_time = 0.0

    idx = collect(1:m)
    shuffle!(idx)
    i = 1

    gw = ones(length(w))

    while norm(gw) > tol && iter < max_iter
        #γ = α/(1 + α*iter)
        gw = g(w,idx[i])
        w_new = w - α * gw
        w = w_new
        gw = g(w,idx[i])
        if i < m
            i = i+1
        else
            i = 1
        end
        iter += 1
        elapsed_time = time() - start_time
        if elapsed_time > max_time
          break
        end
    end
    return w, iter, elapsed_time
end
```


Stochastic Gradient Descent/Ridge and Logistic Regression (paper)
=================================


```julia
using Distributions
using Optim

function sgd(X::Matrix{Float64}, y::Vector{Float64}, beta_init::Vector{Float64}, link_fun::Function, lambda::Float64,
    gamma0::Float64, max_iter = 1e6, eval = false)
    #stochastic gradient descent for ride regression and regularized (12-norm) logistic regression
    #learning rate gamma+t = gamma0/(1+gamma0*lambda*t)
    #make sure beta_init will be unchanged
    beta = copy(beta_init)
    (n, p) = size(X)
    t = 0
    beta_new = zeros(Float64, p+1)
    max_epoch = round(Int, ceil(max_iter/n))
    #stores eval
    f = zeros(Float64, max_epoch*n)
    start_time = time()
    elapsed_time = 0.0
    for epoch = 1:max_epoch
        #random permutation
        idx = collect(1:n)
        shuffle!(idx)
        for i = 1:n
            t = t + 1
            #when t = 1, gamma_t = gamma0
            gamma_t = gamma0/(1+gamma0*lambda*(t-1))
            err_term = -y[idx[i]] + link_fun(beta[1] + dot(beta[2:end], vec(X[idx[i],:])))
            beta_new[1] = beta[1] - gamma_t*err_term
            beta_new[2:end] = beta[2:end] - gamma_t*lambda*beta[2:end] - gamma_t*err_term*vec(X[idx[i],:])
            if eval
                f[t] = cal_obj(X, y, beta_new, link_fun, lambda)
            end
            for j = 1:p+1
                beta[j] = beta_new[j]
            end
            elapsed_time = time() - start_time
        end
    end
    if eval
        return beta, f, elapsed_time
    else
        return beta, elapsed_time
    end
end
```


Others
===


```julia
function gradgd(w,x,y)
    m = length(y)
    n = length(w)
    a = zeros(n)
    for j = 1:n
        for i = 1:m
            a[j] += (y[i] - dot(w,x[i,:]))*x[i,j]
        end
    end
    return a
end

function gd(w_init,X,y;max_iter=1000000, α = 1) #X with ones already
    #starting the parameters
    (m,n) = size(X)
    #w = copy(w_init)
    #w_new = zeros(m)
    max_epoch = round(Int, ceil(max_iter/m))
    start_time = time()
    elapsed_time = 0.0

    #now we look through the entire dataset max_epoch times
    for epoch = 1:max_epoch
        gw = gradgd(w_init,X,y)
        w_init = w_init + (α/m) * gw
        #w = w_new
        #α = 0.9*α
        elapsed_time = time() - start_time
    end
    return w_init, elapsed_time
end
```


Averaged SGD
======


```julia
function gradsgd(w,x,y,i)
    m = length(y)
    n = length(w)
    a = (dot(w,x[i,:]) - y[i]) * x[i,:]
    return a
end

function asgd(w_init,X,y;max_iter=10000, α = 1) #X with ones already
    #starting the parameters
    (m,n) = size(X)
    w = w_init
    w_new = zeros(n)
    max_epoch = round(Int, ceil(max_iter/m))
    start_time = time()
    elapsed_time = 0.0
    r = 0.0
    v = zeros(n)
    iter = 0.0

    #now we look through the entire dataset max_epoch times
    for epoch = 1:max_epoch
        idx = collect(1:m)
        shuffle!(idx)
        for i = 1:m
            #γ = α/(α+iter)
            iter += 1
            gw = gradsgd(w,X,y,idx[i])
            w_new = w - α * gw
            r_new = r + α
            v_new = (r/r_new) * v + ((r_new - r)/r_new) * w
            for j = 1:n
                w[j] = w_new[j]
                v[j] = v_new[j]
            end
            r = r_new
        end
        elapsed_time = time() - start_time
    end
    return v, elapsed_time
end
```


Testing
====


```julia
X = [ones(1000) rand(1000,3)]; w = [-1.0, 2.0, 5.786, 3.98]; y = X*w + 0.1*rand(1000); w_init = zeros(4);
```


```julia
asgd(w_init,X,y,α = .9), w
```




    (([-0.932864,1.99275,5.7641,3.97104],0.0048367977142333984),[-1.0,2.0,5.786,3.98])




```julia
sgdm(w_init,X,y,α = .85), w
```


    (([-0.989569,2.07126,5.99631,3.94538],109,0.0001289844512939453),[-1.0,2.0,5.786,3.98])
