# 1. Bifurcation Diagram
```python
import numpy as np
import matplotlib.pyplot as plt

interval = (2.8, 4)  # start, end
accuracy = 0.0001
reps = 600  # number of repetitions
numtoplot = 200
lims = np.zeros(reps)

fig, biax = plt.subplots()
fig.set_size_inches(16, 9)

lims[0] = np.random.rand()
for r in np.arange(interval[0], interval[1], accuracy):
    for i in range(reps - 1):
        lims[i + 1] = r * lims[i] * (1 - lims[i])

    biax.plot([r] * numtoplot, lims[reps - numtoplot :], "b.", markersize=0.02)

biax.set(xlabel="r", ylabel="x", title="logistic map")
plt.show()
```

# 2. Lyapunov Exponent
```python
#Lyapuonv Exponent
import numpy as np
import matplotlib.pyplot as plt

def logistic(r, x):
    return r * x * (1 - x)

n = 10000 #관찰할 r과 각 r의 상태 x 생성
r = np.linspace(-100, 100, n)
r = np.tanh(r)/8
r += 3.875

x = 0.1 * np.ones(n)

step = 100000 # step 수 및 diagram에 나타낼 state 개수(last) 설정
last = 100000

lyapunov = np.zeros(n)

fig = plt.figure()
subplot = fig.add_subplot(2, 1, 1)
for n in range(step):
    x = logistic(r, x)
    lyapunov += np.log(abs(r - 2 * r * x)) #MLE 계산

lyapunov = lyapunov / step

plt.plot(r, lyapunov, marker=',', ls='', c='black', alpha=1)
plt.axhline(y=0, c='red')
plt.xlim(3.5, 4)
plt.title("Lyapunov exponent")
plt.tight_layout()
```

# 3. Cobweb Plot
```cpp
import numpy as np
import matplotlib.pyplot as plt

def logistic(r, x):
  return r*x*(1-x)

def plot_system(r, x0, n, ax=None):
  t = np.linspace(0, 1)
  ax.plot(t, logistic(r, t), 'k', lw=2)
  ax.plot([0,1], [0,1], 'k', lw=2)

  x= x0
  for i in range(n):
    y = logistic(r, x)
    ax.plot([x, x], [x, y], 'k', lw=1)
    ax.plot([x, y], [y, y], 'k', lw=1)
    ax.plot([x], [y], 'ok', ms=5, alpha=(i+1)/n)
    x=y
  ax.set_xlim(0, 1)
  ax.set_ylim(0,1)

fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(12, 6), sharey=True)

plot_system(2, .1, 100, ax=ax1)
plot_system(3.1, .1, 100, ax=ax2)
plt.show()
```
# 4. Michael Barnsley, Fractal.
```py
int r
X = [0]; Y = [0] 
for n in range(10000): 
    r = RR.random_element (0,100) # 0, 100 사이 난수
    if r < 86.0: 
        x = 0.85*X[n-1] + 0.04*Y[n-1]; y = -0.04*X[n-1] + 0.85*Y[n-1]+1.6
    elif r < 93.0: 
        x = 0.2*X[n-1] - 0.26*Y[n-1]; y = 0.23*X[n-1] + 0.22*Y[n-1] + 1.6
    else: 
        x = -0.15*X[n-1] + 0.28*Y[n-1]; y = 0.26*X[n-1] + 0.24*Y[n-1] + 0.44
    X.append(x);Y.append(y) # 초기값.append(변수) : 좌표값 대입
scatter_plot([(X,Y)],edgecolor='green',facecolor='green', marker = '.') 
#edgecolor : 점 테두리. facecolor : 점 안. marker : 점 모양
```
