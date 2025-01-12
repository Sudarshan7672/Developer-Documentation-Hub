# Redux
Used for managign the state using the centralized states.

## Terms Used In Redux Toolkit
### 1. Store
A centralized store holds the whole state tree of your application.

### 2.Reducers
Function that take the current state and the action as agruments, and return a new state result. In other words  `(state, action)=>newState`

### 3. Action 
It is Plain Javascript object that has a type field.

### 4.Slice
Collection of Redux reducer logic and actions for single feature together in single file.
