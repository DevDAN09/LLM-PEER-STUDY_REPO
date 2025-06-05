[사이트 링크](https://usehooks.com/uselocalstorage)

#### 한줄 요약
로컬 스토리지를 사용하는 훅

> Store, retrieve, and synchronize data from the browser’s localStorage API with useLocalStorage
#### 설명 
로컬스토리지의 저장된 데이터와 함께 컴포넌트의 상태를 동기화 하는 편리한 방법을 제공하는 훅
컴포넌트가 마운트 될때, 값을 읽고, 업데이트 될 경우 저장소 또한 업데이트
다른 탭이나 창에서 로컬 스토리지의 변경사항을 수신 컴포넌트의 상태를 최신을 유지

> The useLocalStorage hook provides a convenient way to synchronize the state of a component with the data stored in local storage. It automatically reads the initial value from local storage when the component mounts and updates the local storage whenever the state changes. Additionally, it listens for changes in local storage made by other tabs or windows and keeps the component’s state up to date.

#### 파라미터

| 이름        | 타입     | 설명                         |
| --------- | ------ | -------------------------- |
| key       | string | 로컬스토리지 키 값                 |
| initValue | any    | 값이 없는 경우, 키값과 함께 전달하는 초기 값 |

#### 리턴 값

| Name           | Type     | Description                                                              |
| -------------- | -------- | ---------------------------------------------------------------------- |
| localstate     | any      | 로컬 스토리지에 저장된 현재 상태의 값                                                    |
| handleSetState | func 로컬 스토리지의 값의 상태를 설정하는 함수, 새로운 값 또는 새로운 값을 반환하는 함수를 전달, 값은 제공된 키의 값에 전달됨 달,  수를  |
