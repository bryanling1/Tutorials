#React

## Table of contents
- [Table of contents](#table-of-contents)
- [Previous Props Hook](#previous-props-hook)
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


