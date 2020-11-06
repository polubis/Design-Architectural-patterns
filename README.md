# Design-Architectural-patterns

## Some good rules / principles

### DRY - Don't Repeat Yourself

<img src="https://www.flaticon.com/svg/static/icons/svg/2654/2654602.svg" height="48" width="48">

*"DRY is a guideline, not a religion. Those who take it to the point of DRY above all else, have taken it too far."*

Repeating the same code in the application may considerably take longer to make the necessary changes and may cause bugs - but not always.

#### Styles

> BAD - duplication in css causes issues with changing layout - designer change his mind
```scss
// in modal.scss
h2 {
  font-family: $fontPrimary;
  text-transform: none;
  font-size: 14px;
  font-weight: $medium;
  line-height: 24px;
}

// in header.scss
h2 {
  font-family: $fontPrimary;
  text-transform: none;
  font-size: 14px;
  font-weight: $medium;
  line-height: 24px;
}

// ...
// in button.scss
// in prompt.scss
// in input.scss
// in textarea.scss
// in datepicker.scss
```

> ACCEPTABLE - duplication only in two files
```scss
// in modal.scss
h2 {
  font-family: $fontPrimary;
  text-transform: none;
  font-size: 14px;
  font-weight: $medium;
  line-height: 24px;
}

// in header.scss
h2 {
  font-family: $fontPrimary;
  text-transform: none;
  font-size: 14px;
  font-weight: $medium;
  line-height: 24px;
} 
```

> OK - mixins used to avoid any code duplication - we can compose our css rules
```scss
@mixin text {
  font-family: $fontPrimary;
  text-transform: none;
}

@mixin label {
  @include text;
  font-size: 14px;
  font-weight: $medium;
  line-height: 24px;
}

// in modal.scss
h2 {
  @include label;
}

h2 {
  @include label;
} // in header.scss
```

#### Constant files

> BAD - helper function `makePaths` makes code hard to read - also if someone change implementation - app destroyed
```ts
export const [FORGOTTEN_PASSWORD, REGISTER, GET_SELF] = makePaths('Account')(
  'ForgottenPassword',
  'Register',
  'GetCurrentUserData'
);

export const [GET_PATTERNS, EDIT_PATTERN, ADD_PATTERN, GET_PATTERN, DELETE_PATTERN] = makePaths(
  'TemplatePatterns'
)('Search', 'Update', 'Add', 'Get', 'Delete');
```

> ACCEPTABLE - duplication easy to change via code editor
```ts
const FORGOTTEN_PASSWORD = 'Account/ForgottenPassword';
const REGISTER = 'Account/Register';
const GET_SELF = 'Account/GetCurrentUserData';
```

> OK - no duplication, easy code
```ts
const CONTROLLER = 'Account';

const FORGOTTEN_PASSWORD = `${CONTROLLER}/ForgottenPassword`;
const REGISTER = `${CONTROLLER}/Register`;
const GET_SELF = `${CONTROLLER}/GetCurrentUserData`;
```

#### React component guards

> BAD - check user admin / rendering children features are duplicated / hard to read nested ternary operator
```ts
const Admin = ({ children }: Guard.Props) => {
  const { pending, authorized, ...state } = useAuthProvider();

  return pending
    ? null
    : authorized
    ? state.user.roles.includes(UserRole.Admin)
      ? typeof children === 'function'
        ? children(state)
        : children
      : null
    : null;
};

const Protected = ({ children }: Guard.Props) => {
  const { pending, authorized, ...state } = useAuthProvider();

  return pending
    ? null
    : authorized
    ? typeof children === 'function'
      ? children(state)
      : children
    : null;
};

const Unprotected = ({ children }: Guard.Props) => {
  const { pending, authorized, ...state } = useAuthProvider();

  return pending
    ? null
    : authorized
    ? null
    : typeof children === 'function'
    ? children(state)
    : children;
};

// and 5 more guard types 
```

> ACCEPTABLE - some code duplication / but only in two places so who cares
```ts
const Admin = ({ children }: Guard.Props) => {
  const { pending, authorized, ...state } = useAuthProvider();

  return pending
    ? null
    : authorized
    ? state.user.roles.includes(UserRole.Admin)
      ? typeof children === 'function'
        ? children(state)
        : children
      : null
    : null;
};

const Protected = ({ children }: Guard.Props) => {
  const { pending, authorized, ...state } = useAuthProvider();

  return pending
    ? null
    : authorized
    ? typeof children === 'function'
      ? children(state)
      : children
    : null;
};
```

> OK - 2 helpers functions created / guards code easier to read / redundant logic is implemented in helper functions
```ts
const isAdmin = (roles: string[]): boolean => roles.includes(UserRole.Admin);

const renderChildren = (
  children: JSX.Element | Guard.Children.RenderProp,
  state: AuthProvider.State
): JSX.Element => (typeof children === 'function' ? children(state) : children);

const Admin = ({ children }: Guard.Props) => {
  const { pending, authorized, ...state } = useAuthProvider();
  
  return pending
    ? null
    : authorized
    ? isAdmin(state.user.roles)
      ? renderChildren(children, state)
      : null
    : null;
};

const Protected = ({ children }: Guard.Props) => {
  const { pending, authorized, ...state } = useAuthProvider();

  return pending ? null : authorized ? renderChildren(children, state) : null;
};

const Unprotected = ({ children }: Guard.Props) => {
  const { pending, authorized, ...state } = useAuthProvider();

  return pending ? null : authorized ? null : renderChildren(children, state);
};
```

#### Tests

> BAD - **should** and **correctly** keywords not needed
```ts
it('should create component correctly', () => {});
it('should add className correctly', () => {});
it('should combine children correctly', () => {});
it('should save user after success response', () => {});
it('should off loader after success response', () => {});
it('should off redirect after success response', () => {});
it('should show error dialog after failure response', () => {});
```

> ACCEPTABLE - **should** and **correctly** keywords removed
```ts
it('creates component', () => {});
it('adds className', () => {});
it('combines children', () => {});
it('saves user after success response', () => {});
it('hides loader after success response', () => {});
it('redirects after success response', () => {});
it('shows error dialog after failure response', () => {});
```

> OK - tests grouped / duplicated text removed / easy to read
```ts
it('creates component', () => {});
it('adds className', () => {});
it('combines children', () => {});
describe('on success', () => {
  it('saves user', () => {});
  it('hides loader', () => {});
  it('redirects', () => {});
});
describe('on failure', () => {
  it('shows error dialog', () => {});
});
```

#### Worst case - duplicated business logic

There is no **ACCEPTABLE** case. Whenever you duplicate a business logic - you are doing something wrong

> BAD - only difference in view layer - business logic code duplicated. Hard to track bugs
```ts
  const FormVariantA = () => {
    const {
      params: { id }
    } = useRouteMatch<{ id: string }>();

    const history = useHistory();

    const [state, setState] = useState(STATE);

    const handleManagement = useCallback(async (formManagers: Form.Manager[]) => {
      setState({ ...STATE, pending: true });

      try {
        const payload = makePayload(formManagers);

        if (id) {
          await editTemplate(id, payload);
          setState({ ...STATE, id });
        } else {
          const addedTemplateId = await addTemplate(payload);
          setState({ ...STATE, id: addedTemplateId });
        }
      } catch {
        setState({ ...STATE });
      }
    }, []);

    useEffect(() => {
      if (state.id) {
        history.replace(`/app/templates/${TemplateCategory.ALL}/${state.id}`);
      }
    }, [state.id]);
  }
  
  const FormVariantB = () => {
    const {
      params: { id }
    } = useRouteMatch<{ id: string }>();

    const history = useHistory();

    const [state, setState] = useState(STATE);

    const handleManagement = useCallback(async (formManagers: Form.Manager[]) => {
      setState({ ...STATE, pending: true });

      try {
        const payload = makePayload(formManagers);

        if (id) {
          await editTemplate(id, payload);
          setState({ ...STATE, id });
        } else {
          const addedTemplateId = await addTemplate(payload);
          setState({ ...STATE, id: addedTemplateId });
        }
      } catch {
        setState({ ...STATE });
      }
    }, []);

    useEffect(() => {
      if (state.id) {
        history.replace(`/app/templates/${TemplateCategory.ALL}/${state.id}`);
      }
    }, [state.id]);
  }
```

> OK - logic grouped in hook - easy to use between different components
```ts
export const useTemplateManagement = (): Return => {
  const {
    params: { id }
  } = useRouteMatch<{ id: string }>();

  const history = useHistory();

  const [state, setState] = useState(STATE);

  const handleManagement = useCallback(async (formManagers: Form.Manager[]) => {
    setState({ ...STATE, pending: true });

    try {
      const payload = makePayload(formManagers);

      if (id) {
        await editTemplate(id, payload);
        setState({ ...STATE, id });
      } else {
        const addedTemplateId = await addTemplate(payload);
        setState({ ...STATE, id: addedTemplateId });
      }
    } catch {
      setState({ ...STATE });
    }
  }, []);

  useEffect(() => {
    if (state.id) {
      history.replace(`/app/templates/${TemplateCategory.ALL}/${state.id}`);
    }
  }, [state.id]);

  return [state, handleManagement];
};

const FormVariantA = () => {
   useTemplateManagement();
}

const FormVariantB = () => {
  useTemplateManagement();
}
```

### KISS

<img src="https://www.flaticon.com/svg/static/icons/svg/75/75559.svg" height="48" width="48">

*"The code is so simple that even a cockroach can understand it."*

Some of the articles, especially those with examples from js, describe kiss as differences between the syntax of different standards - this is a misunderstanding.

#### This is not KISS issue - just different syntax

> OK
```ts
function calc(a, b) {
   return a + b;
}
```

> OK
```ts
const calc = (a, b) => a + b;
```

#### Overcomplicated code

> BAD - **closures / currying** are really good - but not in this case - this is used without any purpose
```ts
  const add = a => b => c => a + b + c + d;
  add(5)(5)(5)(5) // 20
```

> OK - simple intuitive code, generic approach
```ts
  const add = (...args) => args.reduce((acc, arg) => acc + arg, 0);
  add(5,5,5,5) // 20
```

### Code Against Interfaces, Not Implementations

*"The more loosely coupled your code is, the less likely it is that a change in one place will affect code in another."*

*"You can replace the implementation with a better one without breaking anything."*

*"Focusing on what code doesn, not how code is implemented"*.

#### Issue example

> BAD - rely on **implementation**
```ts
class User {
   constructor(public name: string, public description: string) {}
}

// file.ts
const user = new User('piotr', 'I like programming');

// second-file.ts
const user = new User('piotr', 'I like programming');

// third-file.ts
const user = new User('piotr', 'I like programming');
```

> OK - rely on `interface`
```ts
interface IUser {
  name: string;
  description: string;
}

class User implements IUser {
   constructor(public name: string, public description: string) {}
}

// file.ts
const user: IUser = new User('piotr', 'I like programming');

// second-file.ts
const user: IUser = new User('piotr', 'I like programming');

// third-file.ts
const user: IUser = new User('piotr', 'I like programming');
```

#### Coupled code issue - problems with refactor / implementation change

// BAD - class `Car` rely on implementation - if `DriveManager` implementation changes, `n` files behaviour changed
```ts
  class Car {
    drive = new DriveManager();
  }
  
  // file.ts
  const car = new Car();
  // file2.ts
  const car = new Car();
  // file3.ts
  const car = new Car();
```

// OK - class `Car` rely on `interface` - if `DriveManager` implementation changes, only 1 file affected
```ts
 interface DriveAble {
   drive(): void;
 }
 
 class Car {
    constructor(public drive: DriveAble){}
 }
 
 // file.ts
  const car = new Car(new DriveManager());
 // file2.ts
  const car = new Nissan(new HondaDriveManager());
 // file3.ts
  const car = new Volvo(new VolvoDriveManager());
```

#### React components issue

> BAD - if you change ButtonType enum or implementation inside button component - ui will be broken
```ts
enum ButtonType = {
  PRIMARY = 'primary',
  SECONDARY = 'secondary'
}

export const Button = ({ type }) => {
  const theme = useTheme();

  return (
    <button className={`btn ${theme ? theme : ButtonType[type]}`}></button>
  )
}

// parent-component.ts
<Button type={ButtonType.PRIMARY}></Button>
// second-parent-component.ts
<Button type={ButtonType.SECONDARY}></Button>
// third-parent-component.ts
<Button></Button>
```

> OK - rely on interface. Low risk of spoiling something during changes. Much easier to refactor, better performance, lower bundle size. Code is tree-shakable.
```ts
type BaseButtonProps = React.DetailedHTMLProps<
    React.ButtonHTMLAttributes<HTMLButtonElement>,
    HTMLButtonElement
>;

interface ButtonProps extends BaseButtonProps {}

export type Button = (props: ButtonProps) => JSX.Element;

export const PrimaryButton: Button = (props) => <button { ...props } className={`btn primary ${props.className}`}></button>

export const SecondaryButton: Button = (props) => <button { ...props } className={`btn primary ${props.className}`}></button>

export const ThemedButton: Button = (props) => {
    const theme = useTheme();

    return (
        <button { ...props } className={`btn primary ${props.className}`} style={theme}></button>
    )
}

// parent-component.ts
<PrimaryButton />
// second-parent-component.ts
<SecondaryButton />
// third-parent-component.ts
<ThemedButton />
// fourth-parent-component.ts
<MyOwnButtonWhichImplementsButtonInterface />
```

### SOLID

### Composition over inheritance
