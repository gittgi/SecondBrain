# BFS

```table-of-contents
```

##  BFS 탐색 방법

- [DFS](DFS.md)는 연결되어 있는 모든 노드를 정확히 한번씩 방문하고 싶을 때 사용 (연결 요소 전체를 탐색)
- BFS 역시 목적은 똑같으나 탐색 방식 덕분에 부수효과가 생김
- BFS는 시작점 부터 거리가 1인 점, 2인 점, 3인 점..의 순서로 방문하는 것을 뜻함
	- 큐의 front 값을 꺼내온다.
	- 그 주변 애들을 큐에 넣는다.
	- 반복
```python
que = deque()

que.append(1)
while len(que) > 0:
	cur = que[0]
	que.popleft()

	# 여기서 필요한 연산
	cnt += 1 

	for nxt in v[cur]:
		if visited[nxt]:
			continue
		que.append(nxt)
		# 중복으로 들어가는 걸 방지하기 위해 넣으면서 같이 방문처리
		visited[nxt] = True
```

- dfs도 되고 bfs도 되는 문제에서는 (연결요소 단순 탐색) 구현이 간결한 직관적인 dfs를 사용하는 경우가 많다.
- 다만 다음 두 경우에는 bfs 사용
	- 재귀 깊이가 너무 깊어져서 오버헤드가 큰 경우(파이썬)
	- 삼성 코딩테스트처럼 스택 메모리 제한이 따로 명시된 경우 (지역변수 + 함수 메모리) (que의 경우에는 heap 메모리 사용)


### BFS의 부수적인 효과

- 거리 순서로 돌다 보니까, 각 요소까지의 최단거리에 도착하게 됨 -> 최단 거리를 구할 때 활용
- BFS를 사용하는 문제 => 최단거리를 구하는 문제



## BFS 결과물

-  **BFS => 시작 노드 부터 나머지 모든 노드까지의 최단거리를 구하는 알고리즘**


## 최단거리를 구하는 세가지 방법

1. dist 배열 만들기 (보편적인 방법, 모든 노드의 최단거리가 필요하다면 이렇게)
```python
que = deque()
dist = [-1 for i in range(n+1)]

que.append(1)
dist[1] = 0
visited[1] = True
while len(que) > 0:
	cur = que[0]
	que.popleft()

	# 여기서 필요한 연산
	if cur == 12:
		print(dist[cur])

	for nxt in v[cur]:
		if visited[nxt]:
			continue
		que.append(nxt)
		# 중복으로 들어가는 걸 방지하기 위해 넣으면서 같이 방문처리
		visited[nxt] = True
		dist[nxt] = dist[cur] + 1
```

2. que에 노드번호와 거리를 같이 넣어주는 방법
3. 큐의 사이즈를 이용해 끊어서 계산하기
```python
que = deque()
que.append(1)

visited[1] = True
while len(que) > 0:
	# 해당 거리만큼 애들끼리만 확인
	sz = len(que)
	for _ in range(sz):
		cur = que[0]
		que.popleft()

		if cur == 12:
			print(d)
			
		for nxt in v[cur]:
			if visited[nxt]:
				continue
			que.append(nxt)
			# 중복으로 들어가는 걸 방지하기 위해 넣으면서 같이 방문처리
			visited[nxt] = True
	
	# 같은 길이에 있는 애들 다 끝나면 다음 길이
	d += 1

```
- 확장이 가능 (거리가 3 이하인 점 찾기)
```python

que = deque()
que.append(1)

visited[1] = True
for i in range(3):
	# 해당 거리만큼 애들끼리만 확인
	sz = len(que)
	for _ in range(sz):
		cur = que[0]
		que.popleft()

		if cur == 12:
			print(d)
			
		for nxt in v[cur]:
			if visited[nxt]:
				continue
			que.append(nxt)
			# 중복으로 들어가는 걸 방지하기 위해 넣으면서 같이 방문처리
			visited[nxt] = True
	
	# 같은 길이에 있는 애들 다 끝나면 다음 길이
	d += 1
```
- 혹은 한 스텝을 함수로 만들어서 활용하기도 가능