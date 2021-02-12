#React

## Table of contents
[Getting Previous Props](#getting-previous-props)
## Getting previous props
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


