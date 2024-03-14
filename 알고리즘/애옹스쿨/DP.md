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


### 바텀 업 DP

#### 바텀 업 DP란?

- 피보나치를 트리 형태로, 재귀 형태로 푸는 것은 매우 어색한 방식
	- 중복이 많고 불필요
	- 보통은 0 1 1 2 3 ... 순서대로 더해가면서 다음 거를 구하는게 더 효율적
```python
dp[0] = 0
dp[1] = 1
for i in range(2, n):
	dp[i] = dp[i-1] + dp[i-2]
```

- ##### 퇴사 2
	- 탑 다운 dp에서는 recur(1)을 구하기 위해서 재귀로 쭉쭉 올라가서 recur(7)을 확인하고 돌아오면서 dp(n)에 저장했었음 
	- 즉 recur(1) = max(dp(4) + arr(1), dp(2))
	- 이를 **순서를 바꿔서**, dp(7) 부터 확인하고 dp 배열을 채워나가는 것이 바텀 업 
		- 7일 부터는 돈을 벌 수 없고, 6일 역시 마찬가지
		- 5일에는 15원을 벌 수 있고
		- 4일에는 25원 + 5일에 버는 돈 또는 일을 안하고 5일에 버는 돈 
		- 3일에는 일을 하고 그 다음 시간에 돈벌기 or 오늘 일안하고 내일 일했을 떄 돈
```python
dp = [-10000000000 for i in range(n + 5000)]
# 방향을 반대로 해주니, 재귀가 필요가 없어졌다
for i in range(n)[::-1]:
	# max(오늘 일했을 때 값, 오늘 일 안했을 때 값)
	dp[i] = max(dp[i+arr[i][0]] + arr[i][1], dp[i+1])

print(dp[0])
```

>[!Important]
> - 즉, 바텀 업은 탑 다운과 개념은 같지만, **방향이 반대**인 것!!

##### RGB 거리 2
```python
dp[0][0] = arr[0][0]
dp[0][1] = arr[0][1]
dp[0][2] = arr[0][2]

for i in range(1, n):
	for j in range(3):
		for k inr range(3):
			if j == k:
				continue
			# 직전 집 중에 작은 것 + 자신 (같은 라인은 제외)
			dp[i][j] = min(dp[i][j], dp[i - 1][k] + arr[i][j])
print(min(dp[n-1]))
```


#### DP 풀이 과정
1. 이 문제를 DP로 풀어야겠다는 힌트
	- 경우의 수를 구하라
	- 최대 / 최소를 구하라
	- 익숙한 DP 문제
2. DP 테이블 설계
	- 테이블의 차원 등
3. 점화식 구하기
	- DP(x) = DP(??) + DP(???) + arr(???)
4. 반복문을 이용해 구현

- 경험에 영역... 연습을 많이 해야 한다..

#### 탑 다운 to 바텀 업

##### 퇴사 2
```python
# 탑 다운 버전
def recur(cur):
	if cur > n:
		return -1000000
	if cur == n:
		return 0
		
	if dp[cur] != -1:
		return dp[cur]

	dp[cur] = max(recur(cur + arr[cur][0]) + arr[cur][1]), recur(cur+1)
	return dp[cur]

# recur(cur) == dp[cur]
# cur > n : -100000000
# cur == n: 0
# dp[cur] = max(dp[cur + arr[cur][0]] + arr[cur][1], dp[cur+1])
# -> dp[cur]을 구하기 위해서는 dp[cur + arr[cur][0]] + arr[cur][1]와 dp[cur+1]를 먼저 알아야 함 -> 거꾸로 채워주기

dp = [-1000000 for i in range(n + 10000)] # 뒤에 있는 걸 참조할 것이기 때문에, 충분히 크게 만들어서 인덱스 에러를 방지 -> if cur > n : return -1000000를 표현할 수 있도록 구현한 것
dp[n] = 0 # if cur == n: return 0 를 구현한 것
for i in range(n)[::-1]
	dp[i] = max(dp[i + arr[i][0]] + arr[i][1], dp[i++1])

```


##### RGB 거리
```python

def recur(cur, prv):
	if cur == n:
		return 0
	if and dp[cur][prv] != -1:
		return dp[cur][prv]
	ret = 1000000000
	for i in range(3):
		if i == prev:
			continue
		ret = min(ret, recur(cur + 1), i)
	dp[cur][prv] = ret
	return ret
print(recur(0, 3)) # dp 배열을 [[-1, -1, -1, -1]...] 로 짜는 이유는 맨 처음에는 0, 1, 2 다 택할 수 있게 하기 위함

# dp[n][?] = 0
# dp[cur][prv] = min(dp[cur+1][i] + arr[cur][i]) (i != prv)

dp[[0, 0, 0, 0] for i in range(n+1)]
for i in range(n)[::-1]:
	for j in range(4):
		dp[i][j] = 1000000000
		for k in range(3):
			if j == k:
				continue
			dp[i][j] = min(dp[i + 1][k] + arr[i][k])

print(dp[0][3])
```


>[!Important]
> - 탑 다운 : 나를 구하기 위해서는 쟤랑 쟤가 필요해 -> 재귀로 구하러 감
> - 바텀 업 : 나를 구하기 위해서 필요한 쟤랑 쟤를 쓰자 -> 이미 역순으로 구해져 있음

- 재귀함수에서 점화식 뽑아내기 (가지치기, 기저에 있는 걸로 초기화)
- 반복문으로 채우기 (점화식을 확인해서, 채우는 순서 주의해서 유추)



#### well-known DP
- LIS, LCS, 배낭

##### 2 x n 타일링
- 1 * 2 로 시작하거나 2 * 2 시작하는 경우, 2가지로 나눔
	- 겹치지 않고 전체를 커버할 수 있다면 어떻게 나눠도 좋다
- dp(n) == 2 * n 을 채우는 경우의 수
- 만약 1 * 2로 시작 했다면 -> dp(n) =  dp(n- 1)
- 만약 2 * 2로 시작했다면 -> dp(n) = dp(n-2)
- 따라서 dp(n) = dp(n- 1) + dp(n-2) 로 점화식을 세울 수 있다.

> [!Tip] 점화식 TIP
> - DP(입력) = 출력 으로 생각하면 잘 되는 경우가 많다
> - ex ) DP[가로 길이] = 경우의 수
> 


##### 가장 긴 증가하는 부분 수열 (LIS)
- 방법 1 : `DP[i]` == i번째 수 까지만 존재할 때의 가장 긴 증가하는 부분 수열 (어려움)
- 방법 2 : `DP[i]` == i번 째 수로 끝나는 정답 (i번 째를 무조건 포함)(추천)
- 20 을 기준으로 했을 때, 이전에 20보다 크거나 같은 값들은 20으로 끝나는 부분수열에 붙일 수 없음
	- 따라서 20보다 작은 값으로 끝나는 부분 수열 중에서 가장 큰 값 + 1을 통해 20으로 끝나는 부분 수열을 구할 수 있음
```python

dp = [1 for i in range(n)]
for i in range(n):
	for j in range(i):
		if arr[j] >= arr[i]:
			continue
		dp[i] = max(dp[i], dp[j] + 1)


```


##### LCS
- `dp[i][j]` = 첫 수열의 `[:i]`와 두번째 수열의 `[:j]` 의 LCS가 나오도록 만들기
- 결과적으로 `dp[n][m]`울 구하면 답

- ?????? 과 !!!!!!!!!!!!!!!!!의 LCS 가 10이라면, ???????A 와 !!!!!!!!!!!!!!!!!A의 LCS는 11일 것 
- ????????A와 !!!!!!!!!!!!!!!B 의 경우에는 ???A 와 !!!!!!!!, ???? 와 !!!!!!!B, 그리고 ?????와 !!!!!!!!!!!!!의 LCS중 제일 큰 것일 것
- 이 두 사실을 가지고 2차원 dp를 채울 점화식을 세울 수 있음
- `arr1[i]`과 `arr2[j]`가 같은 경우 -> 첫 사실로 인해서 `dp[i][j]` = `dp[i-1][j-1] + 1`
- 다른 경우 -> 두번째 사실로 인해서 `dp[i][j]` = `max(dp[i-1][j], dp[i][j-1])` (`dp[i-1][j-1]은 어차피 dp[i][j-1]에 포함됨)
```python
# 패딩
s = "#" + input()
p = "#" + input()
dp = [[0 for i in range(len(p))] for j in range(len(s))]
for i in range(1, len(s)):
	for j in range(1, len(p)):
		if s[i] == p[j]:
			dp[i][j] = dp[i-1][j-1] + 1
		else:
			dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])

print(dp[len(s) - 1][len(p) - 1])
		
```


#### 토글링
- 바텀 업에서 사용하는 테크닉
- 메모리 사용량을 극단적으로 줄여줌
- `내려가기`(골드 5) 문제에서 사용
- 핵심은 **당장에 `dp[i]`를 구하기위해 필수적인 애들만 남기는 것** -> 필요 없는 부분을 덮어쓰기로 재활용하기

##### RGB 거리 2
```python
n = int(input())
dp = [0, 0, 0](0,%200,%200)
for i in range(n):
	arr = list(map(int, input().split()))
	for j in range(3):
		dp[i%2][j] = 10000000
		for k in range(3):
			if j == k:
				continue
			dp[i%2][j] = min(dp[i%2][j], dp[(i+1)%2][k] + arr[j])
			
print(max(dp[(n-1) % 2]))

```





#### 역추적
- 최대 얼마를 버냐? -> 그렇게 하기 위해서는 어떻게 골라야하냐, 어떤걸 골라야 하나? 등
- dp테이블 크기에 맞춰 똑같이 테이블을 만들고, `dp[i][j]` 에 적힌 값이 어디서 유래 되었는지를 적어 놓는 것
- 이후에 해당 테이블을 역추적하면서 확인 가능
##### 가장 긴 증가하는 부분 수열 4
```python
n = int(input())
arr = list(map(int, input().split()))
dp = [1 for i in range(n)]
prv = [-1 for i in range(n)]

for i in range(n):
	for j in range(i):
		if arr[i] <= arr[j]:
			continue
		if dp[i] < dp[j] + 1:
			dp[i] = dp[j] + 1
			prv[i] = j
ans = 0
idx = 0
for i in range(n):
	if ans < dp[i]:
		ans = dp[i]
		idx = i

while idx != -1:
	print(arr[idx])
	idx = prv[idx]
	


```



#### 기타
- n이 10만 20만 넘어가면 파이썬은 바텀업 고려해보기