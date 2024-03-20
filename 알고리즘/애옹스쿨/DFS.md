# DFS

```table-of-contents
```

##  그래프 용어

- 노드(정점) => 객체
- 엣지(간선) => 객체의 관계 정보 
- 간선에 들어가는 정보
	- 방향 : 방향성이 있다면 유향 그래프, 없으면 무향 그래프(양방향)
	- 가중치 : 모든 간선이 같은 가중치라면 굳이 적지 않음
- 연결요소(무향 그래프) : 간선을 통해 연결된 노드 집합

## 그래프 저장 방법

- 입력 받은 그대로 저장 (크루스칼 할 때 사용)
	- 1이랑 연결된걸 다 찾으려면 간선을 전부 다 확인해야 해서 느리다
- 인접 행렬 (특수 케이스에서 사용)
	- x - y를 잇는 간선이라면 `arr[x][y], arr[y][x]`에 1을 채움
	- 이런식으로 저장하면, 간선을 다 보는 것이 아니라 노드를 다 보는 느낌
	- n 제곱 크기의 행렬 필요 => 너무 커서 잘 안쓰임 + 아직 느림
	- dictionay(map)를 사용하면 2n 수준으로 줄일 수 있지만 그게 결국 인접 리스트랑 같음
- 인접 리스트 (대부분 이렇게 사용)
	- `arr[1]` : 1번 노드에서 갈 수 있는 노드들을 저장 
	- 이러한 방식으로 2차원 리스트를 만들면 됨

## DFS

- dfs(1)의 목적 : 1번 노드에서 출발해서 갈 수 있는 모든 노드를 정확히 한번씩 방문하고 방문처리

```python

def dfs(cur):
	visited[cur] = True

	for nxt in v[cur]:
		if visited[nxt]:
			continue

		dfs(nxt)

```




## 연결 요소 정보 얻기

### 연결 요소의 크기 구하기
```python
def dfs(cur):
	global cnt
	visited[cur] = True
	cnt += 1

	for nxt in v[cur]:
		if visited[nxt]:
			continue

		dfs(nxt)
```

```python
def dfs(cur):
	cnt = 1
	visited[cur] = True
	
	for nxt in v[cur]:
		if visited[nxt]:
			continue

		cnt += dfs(nxt)

	return cnt
```
- 참고로 1 => 3과 1 =>2 => 3한거랑 return 값이 다를 수 있기 때문에 메모이제이션은 적용 불가


### 연결 요소의 개수 구하기

- 1 부터 시작해서 방문처리가 안되어 있다면 cnt +1 하고 1과 연결된 애들 전부 방문 처리
- 2, 3, 4 를 보면서 방문 처리가 안되어 있다면 cnt + 1하고 다시 연결된 애들 전부 방문 처리

```python
def dfs(cur):
	visited[cur] = True

	for nxt in v[cur]:
		if visited[nxt]:
			continue

		dfs(nxt)

for i in range(1, n+1):
	if not visited[i]:
		dfs(i)
		cnt += 1

```



## 플러드필

### 단지번호 붙이기

- 2차원리스트를 그래프로 생각
- 상하좌우가 연결되어 있는 노드로 생각
- 즉, 연결 요소의 개수를 구하는 문제

```python

dx = [-1, 1, 0, 0]
dy = [0, 0, -1, 1]
def dfs(x, y):
	global cnt
	visited[x][y] = True
	cnt += 1

	for i in range(4):
		nx, ny = x + dx[i], y + dy[i]
		if not(0 <= nx < n and 0 <= ny < n) or arr[nx][ny] == '0':
			continue
		if visited[nx][ny]:
			continue
			
		dfs(nx, ny)
	

ans = []
for i in range(n):
	for j in range(n):
		if arr[i][j] == '0':
			continue
		if visited[i][j]:
			continue

		cnt = 0
		dfs(i, j)
		ans.append(cnt)

ans.sort()

```



## 추가

- dfs 만 되고 bfs 가 안되는 경우 => 경로 정보를 같이 저장해야 하는 경우 => 이 경우에는 재귀 깊이가 그렇게 깊지 않을 것
- 이 경우 빼고는 bfs로도 풀 수 있기 때문에, dfs의 재귀깊이가 걱정된다면 bfs로 해보자