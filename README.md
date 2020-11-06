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

<img src="https://www.flaticon.com/svg/static/icons/svg/3443/3443998.svg" height="48" width="48">

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

> BAD - class `Car` rely on implementation - if `DriveManager` implementation changes, `n` files behaviour changed
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

> OK - class `Car` rely on `interface` - if `DriveManager` implementation changes, only 1 file affected
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

<img src="https://www.flaticon.com/svg/static/icons/svg/3076/3076553.svg" height="48" width="48">

#### SRP - Single responsiblity principal

<img src="https://www.flaticon.com/svg/static/icons/svg/1021/1021098.svg" height="48" width="48">

*"ALL WE HAD TO DO WAS FOLLOW THE DAMN TRAIN CJ"*

`class`, `function`, `module` should only have one job.

##### One class doing too many things

> BAD - class `User` doing 2 things - loads user data and holds user information
```ts
class User {
    name: string;
    email: string;

    constructor() { }
    
    loadUser(): Promise<User> { 
        return new Promise((resolve) => { 
            resolve({ name: 'Piotr', email: 'piotr1994@gmail.com' } as User);
        })
    }

    setUser(name: string, email: string): void {
        this.name = name;
        this.email = email
    }
}

const user = new User();
user.loadUser().then(({ name, email }) => {
    user.setUser(name, email);
    
    console.log(user.name);
    console.log(user.email);
});
```

> OK - `User` interface only represents model. Repository allows to get data from api or db. `UserController` manages whole logic.
```ts
interface User {
    name: string;
    email: string;
}

type Repository<T> = Partial<{
    load(): Promise<T>;
}>;

class UserRepository implements Repository<User> {
    load(): Promise<User> {
        return new Promise((resolve) => {
            resolve({ name: 'Piotr', email: 'piotr1994@gmail.com' } as User);
        })
    }
}

class UserController {
    user: User | null = null;

    constructor(public repo: UserRepository) { }

    handleLoadUser(): Promise<User> {
        return new Promise(async (resolve) => {
            this.user = await this.repo.load();

            resolve(this.user);
        })
    }
}

const userController = new UserController(new UserRepository());
userController.handleLoadUser().then((user) => {
    console.log(user);
});
```

##### Function doing too many things

> BAD - The function has 3 responsibilities. Manages entire process of adding a user, deals with API processing and payload formatting
```ts
enum UserType {
    ADMIN = 'ADMIN',
    CONTENT_MANAGER = 'CONTENT_MANAGER',
    SUPER_ADMIN = 'SUPER_ADMIN'
}

const handleSaveUser = async (type: UserType) => {
    setIsLoading(true);
    setError('');

    if (type === UserType.ADMIN) {
        try {
            
            await axios.post('/users', userData);
            setError('');
            setIsLoading(false);
        } catch {
            setError('Error while adding user');
            setIsLoading(false);
        }
    } else if (type === UserType.CONTENT_MANAGER) {
        try {
            await axios.post('/users/cm', userData);
            setError('');
            setIsLoading(false);
        } catch {
            setError('Error while adding user');
            setIsLoading(false);
        }
    }
    else if (type === UserType.SUPER_ADMIN) {
        try {
            await axios.post('/users/sa', userData);
            setError('');
            setIsLoading(false);
        } catch {
            setError('Error while adding user');
            setIsLoading(false);
        }
    }

    throw new Error(`No implementation for given ${type} type`);
}
```

> OK - Function only manages entire process - code is easy to understand because of named functions. Also can be reused in other handlers.
```ts
enum UserType {
    ADMIN = 'ADMIN',
    CONTENT_MANAGER = 'CONTENT_MANAGER',
    SUPER_ADMIN = 'SUPER_ADMIN'
}

const initialize = () => {
    setIsLoading(true);
    setError('');
};

const handleSucces = () => {
    setIsLoading(false);
    setError('');
}

const handleError = (error: string) => {
    setIsLoading(false);
    setError(error);
}

const saveUser = (path: string, payload: any) => {
    return new Promise(async (resolve, reject) => {
        try {
            await axios.post(path, payload);
            resolve()
        }
        catch {
            reject('Error occured');
        }
    });
}

const getPath = (type) => {
    const paths = {
        [UserType.ADMIN]: 'users',
        [UserType.CONTENT_MANAGER]: 'cm',
        [UserType.SUPER_ADMIN]: 'sa',
    };
    
    throw new Error(`No path for given ${type} type`);

    return paths[type];
}

const parsePayload = (userData: Object) => ({
    ...userData
    // SOME PARSING LOGIC
})

const handleSaveUser = (type: UserType) => {
    initialize();
    saveUser(getPath(type), parsePayload(userData)).then(handleSucces, handleError);
}
```

##### Module doing too many things

> BAD - module have 2 responsibilities - error handling and display hard coded component - determines component look
```ts
namespace ErrorBoundary {
  export interface Props {
    children: ReactNode;
  }

  export interface Error {
    name: string;
    message: string;
    componentStack: string;
    occuredAt: Date;
  }

  export interface State {
    hasError: boolean;
  }
}

const STATE: ErrorBoundary.State = {
  hasError: false
};

class ErrorBoundary extends Component<ErrorBoundary.Props, typeof STATE> {
  state = STATE;

  readonly errors: ErrorBoundary.Error[] = [];

  static getDerivedStateFromError(): ErrorBoundary.State {
    return { hasError: true };
  }

  componentDidCatch({ name, message }: Error, { componentStack }: ErrorInfo) {
    const error: ErrorBoundary.Error = {
      name,
      message,
      componentStack,
      occuredAt: new Date()
    };

    this.errors.push(error);
  }

  handleReload = () => {
    window.location.reload();
  };

  render() {
    if (this.state.hasError) {
      return (
        <Modal>
          <h5>Oops, something went wrong.</h5>
          <Button onClick={this.handleReload}>RELOAD</Button>
        </Modal>
      );
    }

    return this.props.children;
  }
}

export default ErrorBoundary;

// usage
<ErrorBoundary><Component /></ErrorBoundary>
```

> OK - presentational component is injected via props - only one job handle error
```ts
namespace ErrorBoundary {
  export interface Props {
    children: ReactNode;
    fallback: (state: State) => JSX.Element;
  }

  export interface Error {
    name: string;
    message: string;
    componentStack: string;
    occuredAt: Date;
  }

  export interface State {
    hasError: boolean;
  }
}

const STATE: ErrorBoundary.State = {
  hasError: false
};

class ErrorBoundary extends Component<ErrorBoundary.Props, typeof STATE> {
  state = STATE;

  readonly errors: ErrorBoundary.Error[] = [];

  static getDerivedStateFromError(): ErrorBoundary.State {
    return { hasError: true };
  }

  componentDidCatch({ name, message }: Error, { componentStack }: ErrorInfo) {
    const error: ErrorBoundary.Error = {
      name,
      message,
      componentStack,
      occuredAt: new Date()
    };

    this.errors.push(error);
  }

  handleReload = () => {
    window.location.reload();
  };

  render() {
    return this.state.hasError ? this.props.fallback(this.state) : this.props.children;
  }
}

export default ErrorBoundary;

// usage
<ErrorBoundary fallback={ErrorScreen}><Component /></ErrorBoundary>
```

#### Open/closed principle - Open for extenstion / closed for modification

<img src="https://www.flaticon.com/svg/static/icons/svg/1169/1169906.svg" height="48" width="48">

*"Imagine you have a bad employee. It is much easier to find a new one than to teach someone with bad habits. The person with bad habits is implementation."*

Objects or entities should be open for extension, but closed for modification. The goal is to make the system easy to extend without incurring a high impact of change. Software systems must be allowed to change their behavior by adding new code rather than changing the existing code.

##### Hello world examples

> BAD - there is not option to add new user dynamically - closed for extension
```ts
export const usersManager = () => {
  const users = ['Piotr', 'Pawel', 'Adam'];
  
  return {
    users
  }
};
```

> OK - open for extension - we can add users dynamically via `add` method
```ts
export const usersManager = () => {
  let users = ['Piotr', 'Pawel', 'Adam'];
  
  return {
    users,
    add: (user: string) => {
      users = [...users, user];
    }
  }
};
```

##### Bucket feature

> BAD - only one formatter allowed - if we would like to change formatter behaviour we need to open this file and add changes. Also this can break all current `Bucket` base features because we need this change only in once place
```ts
interface Item {
    id: number;
    name: string;
    price: number;
}

class Bucket {
    items: Item[] = [];

    private _formatPrice(price: number): number {
        return +price.toFixed(2)
    }

    add(item: Item): void {
        this.items = [...this.items, { ...item, price: this._formatPrice(item.price) }];
    }
}
```

> OK - Option to provide custom formatter added. If we need to change formatting style - just adding different formatting via constructor. 
```ts
interface Item {
    id: number;
    name: string;
    price: number;
}

type Formatter = (price: number) => number;

class Bucket {
    items: Item[] = [];

    constructor(private _formatter: Formatter) {}

    add(item: Item): void {
        this.items = [...this.items, { ...item, price: this._formatter(item.price) }];
    }
}
```

##### List component in React

> BAD - current implementation allows to use only one type of list item. If designer add different type i need to change implementation of this component.
```ts
interface ListProps {
  items: { label: string }[];
}

const List = ({ items }: ListProps) => {
  return (
    <ul>
      {items.map((item, i) => (
        <li key={i}>{item.label}</li>
      ))}
    </ul>
  );
};

// first.tsx
<List items={items} />

// second.tsx
<List items={items} />

// third.tsx - ups different list type needed - new <List> component or implementation change required
<List items={items} />
```

> OK - if we need new list type - we just passing Component / Function as a children - no changes required in implementation
```ts
export interface ListItem {
  label: string;
}

export type ListItemComponent = (props: ListItem) => JSX.Element;

export interface ListProps {
  children: ListItemComponent;
  items: ListItem[];
}

const ListItem: ListItemComponent = ({ label }) => <li>{label}</li>;

export const List = ({ children = ListItem, items }: ListProps) => {
  return (
    <ul>
      {items.map((item, i) => (
        <React.Fragment key={i}>{children(item)}</React.Fragment>
      ))}
    </ul>
  );
};

// first.tsx
<List items={items} />

// second.tsx
<List items={items} />

// third.tsx 
<List children={MyOwnChildren} items={items} />
```

#### Liskov substitution principle


<img src="https://www.flaticon.com/premium-icon/icons/svg/2566/2566163.svg" height="48" width="48">

*"Your bathtub has a leak. You call the plumber and he puts the seal on. Then your father comes and says, Lord, who pays for these things. He removes the earlier plumber and the heat sink in the pipes for some reason. Looks like it works but after hour your neighbor has a swimming pool in the apartment."*

Subclass should override the parent class methods in a way that does not break functionality from a client’s point of view.

##### Invalid spread usage

> BAD - spread operator overwrites formatter method
```ts
type Formatter = (value: number) => string;

const objWithSameProperty = {
    formatter: () => {
        // SOME ULTRA IMPORTANT METHOD
    }
}

const applyFormatter = <T>(obj: T): T & { formatter: Formatter } => {
    const formatter: Formatter = (value) => value.toFixed(2);

    return {
        ...obj,
        formatter
    }
}

// FEATURE DESTROYED
const enhancedObj = applyFormatter(objWithSameProperty);
```

> OK - formatter property never changed also developer friendly error displayed in console - no need to throw an error
```ts
type Formatter = (value: number) => string;

const objWithSameProperty = {
    data: null,
    formatter: () => {
        // SOME ULTRA IMPORTANT METHOD
        return '0';
    }
}

const applyFormatter = <T>(obj: T & { formatter?: Formatter }): T & { formatter: Formatter } => {
    const formatter: Formatter = (value) => value.toFixed(2);

    if (!obj.formatter) {
        console.log('Formatter property is already defined for given object');
    }

    return {
        formatter,
        ...obj,
    }
}

// FEATURE DESTROYED
const enhancedObj = applyFormatter(objWithSameProperty);
```

##### Problem with extension

> BAD - `DesktopCalculator` changes implementation of `Calculator` abstract class - feature destroyed
```ts
abstract class Calculator {
    add(): void {
        console.log('Adding operation');
    }

    diff(): void {

    }

    multiply(): void {}
}

class DesktopCalculator extends Calculator {
    add(): void {
        console.log('Adds calculator to DOM');
    }
}

const desktopCalculator = new DesktopCalculator();
desktopCalculator.add();
```

> OK - composition over inheritance. No risk to overwrite
```ts
interface ICalculator {
    add(): void;
    diff(): void;
    multiply(): void;
}

class Calculator implements ICalculator {
    add(): void {
        console.log('Adding operation');
    }

    diff(): void {

    }

    multiply(): void { }
}

class DesktopCalculator {
    constructor(public calculator: ICalculator) {

    }

    add(): void {
        console.log('Adds calculator to DOM');
    }
}

const desktopCalculator = new DesktopCalculator(new Calculator());
desktopCalculator.add();
```

##### Problem in React component

> BAD - `handleMouseOver` never used if someone pass onMouseOver property to component
```ts
export interface InputProps
  extends React.DetailedHTMLProps<React.InputHTMLAttributes<HTMLInputElement>, HTMLInputElement> {
  status?: 'valid' | 'invalid';
}

export const Input = ({ status, ...inputProps }: InputProps) => {
  const handleMouseOver = (e: React.MouseEvent<HTMLInputElement, MouseEvent>): void => {
    // IMPORTANT EVENT HANDLER NEVER BE USED
  }

  const className = [
    csx.input,
    status === 'invalid' ? csx.invalid : '',
    status === 'valid' ? csx.valid : ''
  ].join(' ');

  return (
    <div className={className}>
      <input onMouseOver={handleMouseOver} {...inputProps} />
      {status === 'invalid' && <WarningIcon />}
      {status === 'valid' && <DoneIcon />}
    </div>
  );
};
```

> OK - no risk of overwriting functionality. Inner component implementation handles event and after handle calls passed onMouseOver handler
```ts
export interface InputProps
  extends React.DetailedHTMLProps<React.InputHTMLAttributes<HTMLInputElement>, HTMLInputElement> {
  status?: 'valid' | 'invalid';
}

export const Input = ({ status, ...inputProps }: InputProps) => {
  const handleMouseOver = (e: React.MouseEvent<HTMLInputElement, MouseEvent>): void => {
    // IMPORTANT EVENT HANDLER NEVER BE USED
    if (inputProps.onMouseOver) {
      inputProps.onMouseOver(e);
    }
  }

  const className = [
    csx.input,
    status === 'invalid' ? csx.invalid : '',
    status === 'valid' ? csx.valid : ''
  ].join(' ');

  return (
    <div className={className}>
      <input {...inputProps} onMouseOver={handleMouseOver} />
      {status === 'invalid' && <WarningIcon />}
      {status === 'valid' && <DoneIcon />}
    </div>
  );
};
```

#### Interface segregation principle

<img src="https://www.flaticon.com/premium-icon/icons/svg/2499/2499312.svg" height="48" width="48">

*"You're out shopping. All you need for eggs, cereals and milk. You look at promotions and come out with a whole shopping basket".*

The code should be designed so that it does not require from developer to provide additional unused parameters.

##### Not needed properties in configuration object

> BAD - `Form` class requires `listeners`, `loggers` parameters always
```ts
interface Enumerable {
    length: number;
}

interface FormConfigItem {
    label: string;
    idx: number;
    value: any;
    fns: Function[];
}

class Form {
    constructor(config: Object, listeners: Function[], loggers: Function[]) {}
}

const req = v => !!v;
const minLength = (length: number) => (value: Enumerable) => value.length > length;

const formConfig: FormConfigItem[] = [
    { label: 'Username', idx: 0, value: '', fns: [req, minLength(2)] },
    { label: 'Email', idx: 1, value: '', fns: [req, minLength(2)] },
    { label: 'Password', idx: 2, value: '', fns: [req, minLength(2)] },
    { label: 'Repeated password', idx: 3, value: '', fns: [req, minLength(2)] },
];

const listeners = [];
const loggers = [];

const form = new Form(formConfig, listeners, loggers);
```

> OK - pure constructor takes only what is needed for `Form` class. All additional logic is added via functions
```ts

interface Enumerable {
    length: number;
}

interface FormConfigItem {
    label: string;
    idx: number;
    value: any;
    fns: Function[];
}

class Form {
    private _listeners: Function[] = [];
    private _loggers: Function[] = [];

    constructor(config: Object) { }

    addListeners(listener: Function): void {
        this._listeners.push(listener);
    }

    addLoggers(logger: Function): void {
        this._loggers.push(logger);
    }
}

const req = v => !!v;
const minLength = (length: number) => (value: Enumerable) => value.length > length;

const formConfig: FormConfigItem[] = [
    { label: 'Username', idx: 0, value: '', fns: [req, minLength(2)] },
    { label: 'Email', idx: 1, value: '', fns: [req, minLength(2)] },
    { label: 'Password', idx: 2, value: '', fns: [req, minLength(2)] },
    { label: 'Repeated password', idx: 3, value: '', fns: [req, minLength(2)] },
];


const form = new Form(formConfig);
form.addLoggers(() => { console.log('Hi from logger') });
form.addListeners(() => {
    console.log('Hi from listener')
})
```

##### Not needed properties

> BAD - idx, label properties should be optional
```ts
interface FormConfigItem {
    label: string;
    idx: number;
    value: any;
    fns: Function[];
}

const formConfig: FormConfigItem[] = [
    { label: 'Username', idx: 0, value: '', fns: [req, minLength(2)] },
    { label: 'Email', idx: 1, value: '', fns: [req, minLength(2)] },
    { label: 'Password', idx: 2, value: '', fns: [req, minLength(2)] },
    { label: 'Repeated password', idx: 3, value: '', fns: [req, minLength(2)] },
];
```

> OK - label marked as optional and idx removed
```ts
const req = v => !!v;
const minLength = (length: number) => (value: Enumerable) => value.length > length;

interface FormConfigItem {
    label?: string;
    value: any;
    fns: Function[];
}

const formConfig: FormConfigItem[] = [
    { value: '', fns: [req, minLength(2)] },
    { value: '', fns: [req, minLength(2)] },
    { value: '', fns: [req, minLength(2)] },
    { value: '', fns: [req, minLength(2)] },
];
```

##### Data normaliaztion / structure issues

> BAD - some properties are not needed on component initialization. Also open / closed principle issue - no methods for dynamically add events. Also too many informations in single interface - should be separated.
```ts
interface Settings {
    emitStrategy: 0 | 1;
    events: Partial<{
        onClick: Function;
        onSubmit: Function;
    }>;
    target: HTMLElement;
}

class DOMListener {
    constructor(private _settings: Settings) {}
}

const domListener = new DOMListener({
    emitStrategy: 1,
    target: document.body,
    events: {
        onClick: () => { }
    }
});
```

> OK - Not needed properties from configuration removed. Some interfaces added to improve readability and later usage. Method to dynamically add events added.
```ts
type SettingsEvents = Partial<{
    onClick: Function;
    onSubmit: Function;
}>;

enum EmitStrategy {
    INSTANT = 0,
    DEBOUNCED = 1
}

interface Settings {
    emitStrategy?: EmitStrategy;
    events?: SettingsEvents;
    target: HTMLElement;
}

class DOMListener {
    constructor(private _settings: Settings) { }

    addEvents(events: SettingsEvents): void {
        this._settings = {
            ...this._settings, events: {
                ...this._settings.events,
                ...events,
            }
        }
    }
}

const domListener = new DOMListener({
    target: document.body,
});
```

#### Dependency Inversion principle

*""*

##### Component 

### Composition over inheritance
