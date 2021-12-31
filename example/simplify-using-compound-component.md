# Simplify Components using Compound Component

Suppose you want to design a dynamic form with dynamic layout.

![image](https://user-images.githubusercontent.com/28493237/147811574-39c4c60b-b628-4c09-852d-fda69a55db4c.png)

## Step 1: Think entire form component as single component.

Suppose you build the entire form in single component. When building complex applications we need to **remember the thumb rule that every component should have single responsibility**.

Now let's add the rest of the requirements:

![image](https://user-images.githubusercontent.com/28493237/147811687-4752c27a-6844-4b14-af62-e91234cd0f94.png)

## Step 2: Split components to make every component responsible for one thing.

![image](https://user-images.githubusercontent.com/28493237/147812476-ae0bbcea-7c36-4715-8598-87ddb843239d.png)

When splitting components, we have to make sure it is easy to share states between all child components. The easiest way is to use compound component.

```js
function App() {
  return (
    <Question>
      <Question.Types
        types={[
            { name: 'MCQ', value: 'MCQ', count: 4 },
            { name: 'SAQ', value: 'SAQ', count: 0 },
            { name: 'LAQ', value: 'LAQ', count: 0 }
          ]}
       />
      <Question.Text />
      <Question.Options />
      <Question.Solution />
      <Question.Answer />
      <Question.Submit onSubmit={que => console.log(que)} />
    </Question>
  )
}
```

Above is high-level view of the final form components. **When splitting the components, first split them into high-level view where every component has exactly one thing to care about.**

* Question component will act as container of all of it's sub-components. All states data will be stores here in React ```context``` and all child inside the
question component will use this context to get and update states data.

* Keeping states separate in one place, make other components inside question stateless and sharing state between all it's child becomes very easy.

* The compound component **provides flexibility to user** and user can change the layout of question form by rearranging the child components.

**When splitting components in high-level view, all child can itself contain other child components for UI of that part.**
Once splitting on high-level is done, next comes the part of integrating the low-level view of components.

## Step3: Building high-level view of components.

```js
import * as React from 'react'

const QuestionContext = React.createContext()

function Question(props) {
  const [state, setState] = React.useReducer(
    (state, newState) => ({ ...state, ...newState }),
    { type: 'MCQ', options: ['', '', '', ''], answer: '', solution: '', text: '' }
  );
  
  const onQuestionChange = (text) => setState({ text });
  const onOptionChange = (options) => setState({ options });
  const onTypeChange = (type) => setState({ type });
  const onSolutionChange = (solution) => setState({ solution });
  const onAnswerChange = (answer) => setState({ answer });

  const value = { ...state, onQuestionChange, onOptionChange, onTypeChange, onSolutionChange, onAnswerChange };

  return (
    <QuestionContext.Provider value={value}>
      {props.children}
    </QuestionContext.Provider>
  )
}

function useQuestionContext() {
  const context = React.useContext(QuestionContext)
  if (!context) {
    throw new Error(
      `Question type components cannot be rendered outside the Question component`,
    )
  }
  return context
}

function Types({ types }) {
  const { type, onTypeChange, onOptionChange } = useQuestionContext();
  
  const onChange = (value, count) => {
    onOptionChange(Array(count).fill(''));
    onTypeChange(value);
  };
  
  return (
    <>
      <Label>Question Type</Label>
      {
        types.map((item, index) => (
          <RadioButton
            key={`type-{index + 1}`}
            onChange={() => onChange(item.value, item.count)}
            value={item.value}
            checked={type === item.value}
          >
            {item.name}
          </RadioButton>
        ))
      }
    </>
  );
}


function Text() {
  const { text, onQuestionChange } = useQuestionContext();
  
  return (
    <>
      <Label>Question</Label>
      <TextArea value={text} onChange={onQuestionChange} />
    </>
  );
}


function Options() {
  const { options, onOptionChange } = useQuestionContext();
  
  const onChange = (index, val) => {
    const optList = [...options];
    optList[index] = val;
    onOptionChange(optList);
  };
  
  if(options.length < 1){
    return null;
  }
  
  return (
    <>
      <Label>Options</Label>
      {
        options.map((item, index) => (
          <Input type="text" key={`opt-${index + 1}`} value={item} onChange={(x) => onChange(index, x)} />
        ))
      }
    </>
  );
}


function Solution() {
  const { solution, onSolutionChange } = useQuestionContext();
  
  if(options.length > 0) {
    return null;
  }
  
  return (
    <>
      <Label>Solution</Label>
      <TextArea value={solution} onChange={onSolutionChange} />
    </>
  );
}


function Answer() {
  const { answer, onAnswerChange } = useQuestionContext();
  
  if(options.length < 1) {
    return null;
  }
  
  return (
    <>
      <Label>Answer</Label>
      <Input type="text" value={answer} onChange={onAnswerChange} />
    </>
  );
}


function Submit({ onSubmit }) {
  const { solution, text, options, type, answer } = useQuestionContext();
  
  const onClick = () => {
    onSubmit({ type, options, answer, text, solution });
  };
  
  return (
    <Button onClick={onClick}>Submit</Button>
  );
}

Question.Submit = Submit
Question.Answer = Answer
Question.Solution = Solution
Question.Options = Options
Question.Text = Text
Question.Types = Types

export default Question;
```

Above example is basic overview of compound component usage to design highly flexible component. We split a large component into small components responsible for one single responsibility.
All data required by these child components are provided by parent context. Notice that these child components are totally stateless and hence highly flexible.


## Bonus Step: Convert Compound Component to API

This step will allow you to use same type of data and states in multiple types of component without thinking about it's implementation.
**Trick is to convert all UI code to child component and data will be passed to them as props.**

```js
import * as React from 'react'

const QuestionContext = React.createContext()

function Question(props) {
  const [state, setState] = React.useReducer(
    (state, newState) => ({ ...state, ...newState }),
    { type: 'MCQ', options: ['', '', '', ''], answer: '', solution: '', text: '' }
  );
  
  const onQuestionChange = (text) => setState({ text });
  const onOptionChange = (options) => setState({ options });
  const onTypeChange = (type) => setState({ type });
  const onSolutionChange = (solution) => setState({ solution });
  const onAnswerChange = (answer) => setState({ answer });

  const value = { ...state, onQuestionChange, onOptionChange, onTypeChange, onSolutionChange, onAnswerChange };

  return (
    <QuestionContext.Provider value={value}>
      {props.children}
    </QuestionContext.Provider>
  )
}

function useQuestionContext() {
  const context = React.useContext(QuestionContext)
  if (!context) {
    throw new Error(
      `Question type components cannot be rendered outside the Question component`,
    )
  }
  return context
}

function Types({ types, component }) {
  const { type, onTypeChange, onOptionChange } = useQuestionContext();
  const Child = component;
  
  const onChange = (value, count) => {
    onOptionChange(Array(count).fill(''));
    onTypeChange(value);
  };
  
  return (
    <Child types={types} onChange={onChange} value={type} />
  );
}


function Text({ component }) {
  const { text, onQuestionChange } = useQuestionContext();
  const Child = component;
  
  return (
    <Child onChange={onQuestionChange} value={text} />
  );
}


function Options({ component }) {
  const { options, onOptionChange } = useQuestionContext();
  const Child = component;
  
  const onChange = (index, val) => {
    const optList = [...options];
    optList[index] = val;
    onOptionChange(optList);
  };
  
  if(options.length < 1){
    return null;
  }
  
  return (
    <Child onChange={onChange} options={options} />
  );
}


function Solution({ component }) {
  const { solution, onSolutionChange } = useQuestionContext();
  const Child = component;
  
  if(options.length > 0) {
    return null;
  }
  
  return (
    <Child onChange={onSolutionChange} value={solution} />
  );
}


function Answer({ component }) {
  const { answer, onAnswerChange } = useQuestionContext();
  const Child = component;
  
  if(options.length < 1) {
    return null;
  }
  
  return (
    <Child onChange={onAnswerChange} value={answer} />
  );
}


function Submit({ onSubmit, component }) {
  const { solution, text, options, type, answer } = useQuestionContext();
  const Child = component;
  
  const onClick = () => {
    onSubmit({ type, options, answer, text, solution });
  };
  
  return (
    <Child onClick={onClick} />
  );
}

Question.Submit = Submit
Question.Answer = Answer
Question.Solution = Solution
Question.Options = Options
Question.Text = Text
Question.Types = Types

export default Question;
```

This technique adds an abstraction and flexibility to the designed compound component. It allows the component to be so flexible that you don't have to worry about how UI is created. The compound component acting as an API handles all the data and states. It works similar to High Order Component where you pass components as props and those component don't have to worry about states. As long as child component has same props, it will always work. **This technique improves reusability of your component.**

```js
function App() {
  return (
    <Question>
      <Question.Types
        component={InlineRadioList}
        types={[
            { name: 'MCQ', value: 'MCQ', count: 4 },
            { name: 'SAQ', value: 'SAQ', count: 0 },
            { name: 'LAQ', value: 'LAQ', count: 0 }
          ]}
       />
      <Question.Text component={TextArea} />
      <Question.Options component={TextInputList} />
      <Question.Solution component={TextArea} />
      <Question.Answer component={TextInput} />
      <Question.Submit component={Button} onSubmit={que => console.log(que)} />
    </Question>
  )
}
```
