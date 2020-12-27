# How to set up Redux in a React Project with Typescript

1.  Install redux, react-redux, redux-thunk

2.  Add the following imports to the root file (index.tsx):

        import { createStore, applyMiddleware } from "redux";
        import { Provider } from "react-redux";
        import thunk from "redux-thunk";

3.  Add import for the App component in the root file

        import { App } from "./components/App";

4.  Create the store in the root file

        const store = createStore(reducers, applyMiddleware(thunk));

5.  Wrap the App component in the render function with the Provider component in the root file

        ReactDOM.render(
        <Provider store={store}>
        <App />
        </Provider>,
        document.querySelector("#root")
        );

6.  Create folders in the src folder for reducers and actions

7.  In the reducers folder, create an index.ts (simple version)

8.  Make a dummy reducer so that index.tsx will run without errors

        import { combineReducers } from 'redux';

        export const reducers = combineReducers({
        counter: () => 1
        });

## Actions

9.  In the actions folder, create an index.ts file (simple version)

10. If using an API, import axios

11. Import Dispatch from redux -> this is the interface for the
    Dispatch function so that when dispatch is passed as an argument
    (see below), it takes that type of Dispatch to satisfy Typescript

12. In the actions folder, create a types.ts file.

13. In the types.ts file, create an enum and list all of the action types in there.
    The default setting in an enum is to assign each item a number incrementally, but you don't have to write it.
    This ensures each item is unique.

        export enum ActionTypes {
            fetchTodos,
            deleteTodo
        }

14. In the actions/index.ts file, add the import for the types file

        import { ActionTypes } from './types';

15. Create the first action creator. This example shows how to fetch data from an API.
    Replace url with the actual url, or define url as a variable earlier in the file with the actual url as the value.

        export const fetchTodos = () => {
            return async (dispatch: Dispatch) => {
            const response = await axios.get(url)

            dispatch({
                type: ActionTypes.fetchTodos,
                payload: response.data
            });
        }

        }

16. In the actions/index.ts file, define an interface for the type of data that you expect back from the API. Remember to export it so it can be used in other files. For example:

        export interface Todo {
            id: number;
            title: string;
            completed: boolean;

        }

17. In the axios get request, add a typescript annotation saying it will be of that interface type and an array

        const response = await axios.get<Todo[]>(url)

18. Create an interface to describe the action that will get passed to the dispatch function
    Remember to export it so it can be used in other files.

        export interface FetchTodosAction {
            type: ActionTypes.fetchTodos;
            payload: Todo[];
        }

19. Add in the Typescript type for the action that gets passed to the dispatch function

        dispatch<FetchTodosAction>({

## Reducers

20. Create a new file in the reducers folder for a specific object/thing used in the app.
    For example: todos.ts

21. Import two interfaces from the actions/index.ts file - the object interface and the action type interface(s).

        import { Todo, FetchTodosAction } from "../actions";
        import { ActionTypes } from "../actions/types";

22. Write the reducer.
    The intial state is provided as the first argument to the function. After state:, write the Typescript type, and then initialise it with an empty array.
    The second argument after the comma is the type of action.

        export const todosReducer = (state: Todo[] = [],
            action: FetchTodosAction) => {
                switch (action.type) {
                    case ActionTypes.fetchTodos:
                        return action.payload;
                    default:
                        return state;
                }
        };

23. In the reducers/index.ts file, import the reducer for the specific object, as well as the interface for that object from the
    actions/index.ts file.

        import { todosReducer } from './todos';
        import { Todo } from '../actions';

24. In the reducers/index.ts file, change the combineReducers function to have a key which is the name of the state,
    and the name of the reducer.

        export const reducers = combineReducers({
        todos: todosReducer
        });

25. Now make an interface for the combineReducers function so it knows what state types the reducer should be returning, putting this just below the
    import statements in the reducers/index.ts file.

        export interface StoreState {
            todos: Todo[]
        }

26. Add the above type in to the combineReducers function:

        export const reducers = combineReducers<StoreState>({

## Connecting Components to Redux

27. In the file to connect to the store, import connect from react-redux.

        import { connect } from "react-redux";

28. Also import the interface of the object and the action creator from the actions/index.ts file

        import { Todo, fetchTodos } from "../actions";

29. Also import StoreState interface from reducers/index.ts. The store state describes the entire state in the store.

        import { StoreState } from "../reducers";

30. Create an interface for the Props the object will receive. This includes the array of objects
    and the function of the action creator.

        interface AppProps {
            todos: Todo[];
            fetchTodos(): any;
        }

/////// CHANGE FOR FUNCTIONAL COMPONENT, NOT CLASS COMPONENT

31. In the class initialisation, add the interface as a type after the React.Component.

        class App extends React.Component<AppProps> {

32. Create a mapStateToProps function at the bottom of the file outside of the class.
    MapStateToProps contains the entire state. So the type it is going to receive is the StoreState.
    The return type of the function, in the {}, is the array of items, which needs a type also.

        const mapStateToProps = (state: StoreState): { todos: Todo[] } => {
            return { todos: state.todos };
        };

33. The type annotations can be destructured like so:

        const mapStateToProps = ({ todos }: StoreState): { todos: Todo[] } => {
            return { todos };
        };

34. Export the component using the connect function.
    The name here has to be different to the name of the class, so change the class name to underscore App in the initialisation.

    The connect function takes two function calls.
    The first function call takes the mapStateToProps function and the action creator.
    THe second function call takes the component.

        export const App = connect(mapStateToProps, { fetchTodos })(_App);

35. Call the action creator from the props object.

    In functional components:

        props.fetchTodos();

    In class components:

        this.props.fetchTodos();
