[<img src="http://Pinecoders.com/images/PineCodersLong.png">](http://pinecoders.com)

# Digital Signal Processing In Pine, by [Alex Grover](https://www.tradingview.com/u/alexgrover/#published-scripts)

## Table of Contents

- [Introduction](#Introduction)
- [Digital Signals](#Digital-Signals)
- [Cross Correlation](#Integrator)
- [Convolution](#Convolution)
- [Impulse Function And Impulse Response](#Impulse-Function-And-Impulse-Response)
- [Step Function And Step Response](#Step-Function-And-Step-Response)
- [FIR Filter Design In Pine](#FIR-Filter-Design-In-Pine)
- [Calculating Different Types Of Filters In Pine](#Calculating-Different-Types-Of-Filters-In-Pine)
- [Exponential Averager](#Exponential-Averager)
- [Matched Filter](#Matched-Filter)
- [Differentiator](#Differentiator)
- [Integrator](#Integrator)

## Introduction


[Pine](https://www.tradingview.com/pine-script-docs/en/v4/Introduction.html) is a really lightweight scripting language but it still allows for the computation of basic signal processing processes. In this guide, basic techniques and tools used in signal processing are showcased alongside their implementation in Pine. It is recommended to know the basics of Pine before reading this article. Start [here](https://github.com/pinecoders/pinecoders.github.io/tree/master/learning_pine_roadmap) if you don't.

## Digital Signals

A digital signal is simply a sequence of values expressing a quantity that varies with time. When using Pine, you'll mostly be processing market prices as your main signal. However it is possible to process/generate a wide variety of digital signals with Pine.

### Periodic Signals

A periodic signal is a signal that repeats itself after some time. The image below shows a few different periodic signals:

<p align="center">
<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/7/77/Waveforms.svg/600px-Waveforms.svg.png">
</p>

Periodic signals possess characteristics such as **cycles**, **period**, **frequency** and **amplitude**. The **cycles** are the number of times the signal repeat himself, the **period** represents the duration of one cycle, the **frequency** is the number of cycles made by the signal in one second and is calculated as the reciprocal of the **period** or `1/period`, and finally the amplitude is the highest absolute value of the signal.   

### Generate Periodic Signals

The simplest periodic signal is the **sine wave** and is computed in Pine as follows :

```
//@version=4
//-----
study("Sine Wave")
pi = 3.14159
n = bar_index
period = 20
amplitude = 1
sin = sin(2*pi*1/period*n)*amplitude
//---- Plot
plot(sin, color=color.blue)
```

*as a reminder, `bar_index` is defined as the current number of `close` data points and is just a linear series of values equal to 0, 1, 2, 3, ...*

Other common periodic signals are :

The **triangular wave** computed in Pine as follows :

```
//@version=4
study("Triangular Wave")
//-----
pi = 3.14159
n = bar_index
period = 20
amplitude = 1
triangle = acos(sin(2*pi*1/period*n)*amplitude)
//---- Plot
plot(triangle, color=color.blue)
```

The **square wave** computed in Pine as follows :

```
//@version=4
//-----
study("Square Wave")
pi = 3.14159
n = bar_index
period = 20
amplitude = 1
square = sign(sin(2*pi*1/period*n))*amplitude
//---- Plot
plot(square, color=color.blue)
```

*`sign(x)` is the signum function and yields `1` when `x > 0` and `-1` when `x < 0`*

The **sawtooth wave** computed in Pine as follows :

```
//@version=4
study("Sawtooth Wave")
//-----
n = bar_index
period = 20
amplitude = 1
saw = (((n/period)%1 - .5)*2)*amplitude
//---- Plot
plot(saw, color=color.blue)
```

*`%` represents the modulo which is the remaining of a division and is not related to percentage here*

*Note that those are not the only ways to compute those signals.*

## Cross Correlation

Cross correlation measures the similarity between two signals, preferably stationary with mean ≈ 0. The cross correlation between signal `f` and `g` is often denoted with `f★g`. In Pine cross correlation can be calculated as follows: `cum(f*g,length)` and the the running cross correlation of period `length` as `sum(f*g,length)`.

## Convolution

Convolution is one of the most important concepts in signal processing. Basically, convolution is a combination of two signals to make a third signal. The convolution
is mathematically denoted by the symbol `*`, not to be confused with the multiplication symbol in programming.

For example the convolution of a signal `f` and another signal `g` is denoted `f*g`, in Pine this is done as follow :

```
length = input(14)
//-----
convolve = sum(f*g,length)
```
Or alternatively :

```
length = input(14)
//-----
convolve = 0
for i = 0 to length-1
  convolve := convolve + f[i]*g[i]
```

It can be seen that convolution is similar to a dot-product. In digital signal processing convolution is what allows to create certain filters.


## Impulse Function And Impulse Response

An impulse function represents a series of values equal to 0, except at one point in time where the value is equal to 1. In Pine we can make an impulse function by using the following code :

```
//@version=4
study("Impulse")
length = input(100)
//-----
n = bar_index
impulse = n == length ? 1 : 0
//---- Plot
plot(impulse, color=color.blue)
```
Where the impulse is equal to 1 when `n` is equal to `length`.

The impulse response of a system is a system using an impulse function as its input. The impulse response of a filter is the filter output using an impulse function as its input.



## Step Function And Step Response

The step function, also called *heavy-side step function*, represents a series of values equal to 0 and then to 1 for the rest of the time. The step function is the cumulative sum of the impulse function `step = cum(impulse)` (therefore the impulse function is the differentiation of the step function or `impulse = change(step)`). In Pine we can make a step function by using:

```
//@version=4
study("Step")
length = input(100)
//-----
n = bar_index
step = n > length ? 1 : 0
//---- Plot
plot(step, color=color.blue)
```
Where the step is equal to 1 when `n` is greater than `length`.

The step response of a system is a system using a step function as input. The step response of a filter is the filter output using a step function as input.

## FIR Filter Design In Pine

Because Pine allow for convolution it is possible to design a wide variety of FIR filters. A filter `filter(input)` is equal to `input * filter(impulse)` where `*` denote convolution, or more simply a filter output using a certain input is the convolution between the input and the filter impulse response.

In Pine you can make filters using :

```
filter(input) =>
    sum = 0.
    for i = 1 to length
        sum := sum + input[i-1] * w[i-1]
    sum
```
where `w[i-1]` are the filter coefficients *(or filter kernel)*, note that the sum of `w` must add up to 1 *(this is called total unity)*. It is more convenient for `w` to be expressed as a function `w(x)`.

When we can't have total unity *(The sum of the coefficients doesn't add to 1)* we can rescale the convolution, this is done as follows :

```
filter(input)=>
    a = 0.
    b = 0.
    for i = 1 to length
        w = sin(i/length)
        b := b + w
        a := a + input[i-1]*w
    a/b
```

here `w` does not add to 1, however because we divide the convolution output by the sum of the coefficients *(`b` in the script)* we can get the filter without problems.

You can also look at the following [template](https://www.tradingview.com/script/VttW3bJY-Template-For-Custom-FIR-Filters-Make-Your-Moving-Average/) which allow to design FIR filters.

### Simple Moving Average *(Boxcar)* Filter In Pine

The simple *(or arithmetic)* moving average *(SMA)* sometimes called boxcar filter can be made in different ways with Pine. The standard and more efficient way is to use the integrated `sma(source,length)` function as follows :

```
//@version=4
study("Sma",overlay=true)
length = input(100)
//-----
Sma = sma(close,length)
//---- Plot
plot(Sma, color=color.blue)
```
this code plot the simple moving average of the closing price of period `length`.

Another way is to use convolution with :

```
//@version=4
study("Sma",overlay=true)
//-----
Sma(input,length) =>
    sum = 0.
    for i = 1 to length
        sum := sum + input[i-1] * 1/length
    sum
//---- Plot
plot(Sma(close,100), color=color.blue)
```
`1/length` is the filter kernel of the simple moving average.

An alternative way that allow to use variable integers as period can be made using the following code :

```
//@version=4
study("Sma",overlay=true)
//-----
Sma(input,length) =>
    a = cum(input)
    (a - a[length])/length
//---- Plot
plot(Sma(close,100), color=color.blue)
```

### Gaussian Filter  

The gaussian filter is a filter who possess an impulse response equal to a gaussian function :

<p align="center">
<img src="https://www.mathworks.com/help/examples/fuzzy/win64/GaussianMembershipFunctionExample_01.png">
</p>  

The gaussian function is calculated using the standard formula :

<img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/9d128aef1457349d67843e863bf84aaf24f66ecf">

<br> with *b* = position of the peak and *c* = curve width, by default Pine can return an approximation of a gaussian filter by using the function `alma(input, length, b, c)` with *b* = 0.5.  

A gaussian filter can also be made in Pine via convolution using the following code :   

```
//@version=4
study("Gaussian Filter",overlay=true)
//----
gauss(input, length, width)=>
    a = 0.
    b = 0.
    for i = 1 to length
        w = exp(-pow(i/length - .5,2)/(2*pow(width,2)))
        b := b + w
        a := a + input[i-1]*w
    a/b
//---- Plot
plot(gauss(close, 100, 0.2), color=color.blue)
```

## Calculating Different Types Of Filters In Pine

Filters comes with different types, where the type define how the filter interact with the frequency content of the signal.

### Low-Pass Filters

Low-pass filters are the most widely used type of filters, they remove high-frequency components *(noise)* and thus smooth the signal. By default most of the filter function integrated in Pine *(sma, wma, swma, alma...etc)* comes as low-pass filters.

Low-pass filters posses the following frequency response :

<p align="center">
<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/6/60/Butterworth_response.svg/525px-Butterworth_response.svg.png">
</p>

When the filters coefficients adds up to 1, the filter is a low-pass filter.

### High-Pass Filters

High-pass filters remove low-frequency components of a signal thus keeping higher-frequency components in the signal. They can be made by subtracting the input with the low-pass filter output : `highpass = input - lowpass(input)`. For example a simple moving average high-pass filter can be made in Pine as follows :

```
//@version=4
study("Highpass SMA",overlay=true)
length = input(14)
//-----
hp = close - sma(close,length)
//---- Plot
plot(hp, color=color.blue)
```
High-pass filters posses the following frequency response :

<p align="center">
<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/4/45/Hochpass_1._u._2._Ordnung.svg/600px-Hochpass_1._u._2._Ordnung.svg.png">
</p>

When the filters coefficients adds up to 0, the filter is an high-pass filter.

### Band-Pass Filters

Band-pass filters remove low/high-frequency components of a signal thus keeping mid-frequency components in the signal. They can be made by applying a low-pass filter to an high-pass filter output : `bandpass = lowpass(highpass(input))`. For example a simple moving average band-pass filter can be made in Pine as follows :

```
//@version=4
study("Bandpass SMA",overlay=true)
length = input(14)
//-----
bp = sma(close - sma(close,length),length)
//---- Plot
plot(bp, color=color.blue)
```
High-pass filters posses the following frequency response :

<p align="center">
<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/6/6b/Bandwidth_2.svg/450px-Bandwidth_2.svg.png">
</p>

### Band-Stop Filters

Band-stop filters remove mid-frequency components of a signal thus keeping low/high-frequency components in the signal. They can be made by subtracting an bandpass filter output to an input : `bandstop = input - bandpass`. For example a simple moving average band-stop filter can be made in Pine as follows :

```
//@version=4
study("Bandstop SMA",overlay=true)
length = input(14)
//-----
bs = close - sma(close - sma(close,length),length)
//---- Plot
plot(bs, color=color.blue)
```
Band-stop filters posses the following frequency response :

<p align="center">
<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/e/e0/Acoustics_filter_imp_bstop.jpg/800px-Acoustics_filter_imp_bstop.jpg">
</p>

## Exponential Averager

The exponential averager *(also known as exponential moving average, leaky integrator or single exponential smoothing)* is a widely used recursive filter, this filter is similar to the simple moving average but its computation is way more efficient. This filter posses the following form : `output = α*input+(1-α)*output[1]`.

In Pine we can use the integrated function `ema(input,length)` where `α = 2/(length+1)`

Or as follows :  

```
ea(input,alpha)=>
    s = 0.
    s := alpha * input + (1-alpha) * nz(s[1],close)  
```

with `1 > alpha > 0`.

## Matched Filter

A matched filter is a system who maximize the signal to noise ratio of the output. The impulse response of those filters are equal to the reversed input. Since Pine can use variables as index we can do such filters using the following code :

```
filter(x,y) =>
    sum = 0.
    w = 0.
    for i = 0 to length-1
        w := w + y[length-i-1]
        sum := sum + x[i] * y[length-i-1]
    sum/w
```

where `x` is the input and `y` the signal of interest.

## Differentiator

The first difference differentiator is the most simple of all differentiators and consist in subtracting our current input value with its anterior value. In Pine this can be done with `diff = input - input[1]` or `change(input)`.

## Integrator

The rectangular integrator is simply a running summation which can be made in Pine with the function : `cum(signal)` or :

```
a = 0.
a := nz(a[1],input) + input
```