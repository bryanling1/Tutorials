#React

## Table of contents
- [Table of contents](#table-of-contents)
- [Previous Props Hook](#previous-props-hook)
- [Drag and Drop](#drag-and-drop)
- [Context api](#context-api)
## Previous Props Hook
```jsx
import {useRef} from 'react';

...

function usePrevious<T>(val: T) {
    const ref = useRef<T>();
    useEffect(() => {
        ref.current = val;
    });
    return ref.current as T;
}

const prevProps = usePrevious(props);
```

## Drag and Drop

```jsx
import React, { Component } from 'react';
import ReactDOM from 'react-dom';
import { DragDropContext, Droppable, Draggable } from 'react-beautiful-dnd';

// fake data generator
const getItems = count =>
  Array.from({ length: count }, (v, k) => k).map(k => ({
    id: `item-${k}`,
    content: `item ${k}`,
  }));

// a little function to help us with reordering the result
const reorder = (list, startIndex, endIndex) => {
  const result = Array.from(list);
  const [removed] = result.splice(startIndex, 1);
  result.splice(endIndex, 0, removed);

  return result;
};

const grid = 8;

const getItemStyle = (isDragging, draggableStyle) => ({
  // some basic styles to make the items look a bit nicer
  userSelect: 'none',
  padding: grid * 2,
  margin: `0 ${grid}px 0 0`,

  // change background colour if dragging
  background: isDragging ? 'lightgreen' : 'grey',

  // styles we need to apply on draggables
  ...draggableStyle,
});

const getListStyle = isDraggingOver => ({
  background: isDraggingOver ? 'lightblue' : 'lightgrey',
  display: 'flex',
  padding: grid,
  overflow: 'auto',
});

class App extends Component {
  constructor(props) {
    super(props);
    this.state = {
      items: getItems(6),
    };
    this.onDragEnd = this.onDragEnd.bind(this);
  }

  onDragEnd(result) {
    // dropped outside the list
    if (!result.destination) {
      return;
    }

    const items = reorder(
      this.state.items,
      result.source.index,
      result.destination.index
    );

    this.setState({
      items,
    });
  }

  // Normally you would want to split things out into separate components.
  // But in this example everything is just done in one place for simplicity
  render() {
    return (
      <DragDropContext onDragEnd={this.onDragEnd}>
        <Droppable droppableId="droppable" direction="horizontal">
          {(provided, snapshot) => (
            <div
              ref={provided.innerRef}
              style={getListStyle(snapshot.isDraggingOver)}
              {...provided.droppableProps}
            >
              {this.state.items.map((item, index) => (
                <Draggable key={item.id} draggableId={item.id} index={index}>
                  {(provided, snapshot) => (
                    <div
                      ref={provided.innerRef}
                      {...provided.draggableProps}
                      {...provided.dragHandleProps}
                      style={getItemStyle(
                        snapshot.isDragging,
                        provided.draggableProps.style
                      )}
                    >
                      {item.content}
                    </div>
                  )}
                </Draggable>
              ))}
              {provided.placeholder}
            </div>
          )}
        </Droppable>
      </DragDropContext>
    );
  }
}
```

## Context api
```jsx
import { createContext, useReducer } from 'react';

export const AppContext = createContext({
  user: null,
  addUser: function(user){}
})

export const AppContextProvider = (props) => {
  const [users, dispatch] = useReducer((state, action)=>{
    return [...state, ...action]
  }, [{name: 'Francis', {name:'Mike'}}]) //defaults

  const handlAddUser = (user) => {
    dispatch([user])
  }

  return(
    <AppContext.Provider
      value={{
        users: users,
        addUser: handleAddUser
      }}
    >
      {props.children}
    </AppContext.Provider>
  )
}
export 
```
- Wrap all of `app.js`

Now to use the data:
```jsx
import {useContext} from 'react';
import { AppContext } from '../store/app_context';

const Comp = () =>{
  const appCtx = useContext(AppContext);
  ...
}

```