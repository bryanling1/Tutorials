# Redux

# Table of Contents

# Typescript Typing
Getting types from the store
```ts
import { connect, ConnectedProps } from 'react-redux';
import { StoreState } from '../store/reducers';
import * as actions from '../store/actions';

function mapStateToProps(state:StoreState){
    ...
}

const connector = connect(mapStateToProps, actions);

type PropsFromRedux = ConnectedProps<typeof connector>;

interface StateProps{
    ...
}

type Props = StateProps & PropsFromRedux;

const Component = (props:Props) =>{
    ...
}

export default connector(Component)
```