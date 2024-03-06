# DP

```table-of-contents
```

##  DP (Dynamic Programming, 동적 계획법)

dp dlc => 토글링, 역추적, 비트마스킹, 행렬
### 탑 다운 DP

- 백트래킹이 너무 오래걸림
- 근데 중복 케이스를 많이 보는 중
	- 각 케이스의 정답을 저장해두고, 중복 케이스를 다시 본다면 저장해둔걸 사용하자
		- 정답 저장하는것 => 메모이제이션
- 재귀함수에서 메모이제이션을 이용해 시간복잡도를 줄이는 방법 => 탑다운 DP

#### 피보나치
```python

def f(x):
	if x <= 1:
		return x
	return f(x-1) + f(x-2)

dp = [-1 for i in range(n+1)]


def f(x):
	if x <= 1:
		return x
		
	if dp[x] != -1:
		return dp[x]
		
	dp[x] = f(x-1) + f(x-2)
	return dp[x]
```
- f(4)를 구할 때, f(3) + f(2)
- f(3)을 구할 때 이미 f(2)가 구해짐
- 다시 f(2)를 구하지 않고 그냥 메모이제이션 해뒀다가 불러오기

- 원래 피보나치의 시간 복잡도는 O(2^n) 수준
- 그러나 DP 적용하게 된다면 O(n)수준으로 급감

#### 퇴사문제
- 퇴사 문제
```python

def recur(cur, total):
	global ans
	if cur > n:
		return
	if cur == n:
		ans = max(ans, total)
		return
	recur(cur + arr[cur][0], total + arr[cur][1])
	recur(cur + 1, total)

```

- 퇴사 문제 2
	- n 이 15에서 1500000으로 증가됨 => 시간초과
	- 따라서 DP로 줄이는 연습

1. 백트래킹으로 짠다.
2. 함수 형태를 바꾼다 (리턴하는 방식으로)
3. 메모이제이션 추가

- f(2) = max(f(7) + 20, f(3))
```python

# cur 날짜일때 앞으로 최대로 벌 수 있는 금액
# recur(6) => 앞으로 벌 수 있는 돈이 없기 때문에 0
# recur(5) => 5일에 벌면 15원을 벌 수 있기 때문에 15
def recur(cur):

	# 넘쳤을 때 0을 return하면 안됨 -> 0으로 돌아오면 마치 마지막 날에도 잘 일하고 돈 번 셈이 됨
	# 잘못갔을 때 큰 손해를 보게 만들어서 절대 일한 것으로 카운팅 되지 않도록 하기
	if cur > n:
		return -20000000000
	if cur == n:
		return 0

	# 일했을 때 도착하는 날의 최대값과 일 안하고 다음날의 최대값 사이의 값
	return max(recur(cur + arr[cur][0]) + arr[cur][1], recur(cur + 1))

	# recur(cur + arr[cur][0], total + arr[cur][1])
	# recur(cur + 1, total)
	# VS
	# recur(cur + arr[cur][0] + arr[cur][1])
	# recur(cur + 1)

```

- 기존 백트래킹과 달라진 점
	- total 값이 누적되면서 들어가는지
	- return 되면서 최대값을 찾을 것인지의 차이


- 이제 메모이제이션으로 저장 -> 이미 최대값이 확정되었으면 굳이 또 구할 필요 없이 리턴하기
```python

# (1)
dp = [-1 for i in range(n)]
def recur(cur):

	if cur > n:
		return -20000000000
	if cur == n:
		return 0

	# 만약 이미 구한 것이라면 그게 그냥 최대값 (2)
	if dp[cur] != -1:
		return dp[cur]
	# 만약 아직 안 구한 것이라면 최대값 구하기 
	dp[cur] = max(recur(cur + arr[cur][0]) + arr[cur][1], recur(cur + 1))
	# 최대값 구한거 리턴 (3)
	return dp[cur]
```
- return 식으로 백트래킹을 짰다면, (1), (2), (3) 추가로 dp 완성

#### dp의 시간 복잡도
	1. dp 테이블 n칸을 채워야 한다
	2. 한칸을 채우기 위해 m개의 함수(연산)을 호출해야 한다.
	3. 따라서 dp의 시간 복잡도는 O(m * n)


#### RGB 거리

- 만약 백트래킹으로 짠다면
```python

def recur(cur, total, prv):
	if cur == n:
		ans = min(ans, total)
		return


	for i in range(3):
		if i == prv:
			continue
		recur(cur + 1, total + arr[cur][i], i)

ans = 1 << 60
recur(0, 0, -1)
```

- return 방식으로 짠다면
```python
def recur(cur, prv):
	# 마지막 줄에 왔다면 더이상 드는 비용은 없다
	if cur == n:
		return 0

	ret = 1 << 60
	for i in range(3):
		if i == prv:
			continue
		ret = min(ret, recur(cur + 1, i) + arr[cur][i])

	return ret

print(recur(0, -1))
```
- total에 값을 더해서 다음단계로 넘기는 대신, 더한 뒤에 return으로 반환 (대신 최소값 찾아서)
- 기저에서 찾던 min 값을 return할 때 구해서 return


- 여기서 DP 로 발전
```python
def recur(cur, prv):

	if cur == n:
		return 0

	# prv 값에 따라서 값이 달라질 수 있다? -> prv까지 dp 테이블에 포함
	if dp[cur][prv] != -1:
		return dp[cur][prv]

	ret = 1 << 60
	for i in range(3):
		if i == prv:
			continue
		ret = min(ret, recur(cur + 1, i) + arr[cur][i])

	dp[cur][prv] = ret
	return ret

dp = [[-1 for i in range(3)] for j in range(n)]
```


#### 평범한 배낭

- 배낭 문제

- 백트래킹으로 풀면
```python

def recur(cur, weight, price):
	global ans
	
	if weight > m:
		return
	
	if cur == n:
		ans = max(ans, price)
		return
	

	recur(cur + 1, weight + arr[cur][0], price + arr[cur][1])
	recur(cur + 1, weight, price)


```

- 이걸 return 형식으로 풀면
```python
def recur(cur, weight):

	if weight > n:
		return -100000000000

	if cur == n:
		return 0

	return max(recur(cur + 1, weight + arr[cur][0]) + arr[cur][1], recur(cur + 1, weight))
```

- 이걸 dp
```python
def recur(cur, weight):

	if weight > n:
		return -100000000000

	if cur == n:
		return 0
	if dp[cur][weight] != -1:
		return dp[cur][weight]

	
	dp[cur][weight] = max(recur(cur + 1, weight + arr[cur][0]) + arr[cur][1], recur(cur + 1, weight))
	return dp[cur][weight]

dp = [[-1 for i in range(100010)] for j in range(n)]
print(recur(0, 0))
```
- 시간복잡도는 테이블의 사이즈 * 안에 for문의 갯수(함수 몇번 호출하나)

#### 공룡게임

```python
# cnt1는 몇개 연속으로 선인장이 깔렸는지, cnt2는 몇개 연속으로 2짜리가 나오면 two는 2가 지금까지 존재 했는지
def recur(cur, cnt1, cnt2, two):
	if cnt1 > 2 or cnt2 >= 2:
		return

	if cur == n:
		if two:
			ans += 1
		return

	recur(cur + 1, 0, 0, two)
	recur(cur + 1, cnt1+1, 0, two)
	recur(cur + 1, cnt1 + 1, cnt2 + 1, True)
		


recur(1, 0, 0, False)
```


```python

def recur(cur, cnt1, cnt2, two):
	if cnt1 > 2 or cnt2 >= 2:
		return 0

	# 가능한 걸 찾으면 1을 리턴해서 경우의 수를 하나 늘릴 수 있도록
	if cur == n:
		return two
	# 기저 바로 아래에서 있으면 리턴 / 없으면 구해서 리턴
	if dp[cur][cnt1][cnt2][two] != -1:
		return dp[cur][cnt1][cnt2][two]

	dp[cur][cnt1][cnt2][two] = recur(cur + 1, 0, 0, two) + recur(cur + 1, cnt1+1, 0, two) + recur(cur + 1, cnt1 + 1, cnt2 + 1, 1) %= 100000007
	return dp[cur][cnt1][cnt2][two]

# 인자만큼 dp 테이블 차원이 늘어남
dp = [[[[-1 for i in range(2)] for j in range(2)] for k in range(3)] for l in range(n)]
		


```