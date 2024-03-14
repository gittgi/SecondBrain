# useMemo

```table-of-contents
```

##  useMemo

- [useEffect](useEffect.md)가 [Components](Components.md)가 렌더링 된 후에 실행되는 부수 효과 (side effects)를 다루는 데 사용된다면, useMemo의 경우에는 계산 비용이 높은 연산 결과를 메모이제이션 하여 성능을 최적화하는 데 사용됨
- useMemo의 의존성 배열에 지정된 값이 변경될 때만 계산을 다시 수행하고, 그렇지 않으면 이전에 계산된 값을 재사용
- 따라서 useMemo는 복잡한 계산이나 재계산을 피해야 하는 값의 계산에 주로 사용됨
- useEffect는 비동기적으로 작동할 수 있지만, useMemo는 동기적으로 작동
- useMemo는 렌더링 중에 실행되기 때문에 렌더링을 방해하지 않는 범위 내에서 사용해야 함
- 가장 큰 차이점은 useEffect는 해당 컴포넌트의 렌더링이 완료된 후에 실행, **useMemo는 렌더링 중에 실행됨**