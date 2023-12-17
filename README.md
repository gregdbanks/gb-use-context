 ## Prop Drilling

```javascript
import React from 'react';

// ParentComponent
const ParentComponent = () => {
    const data = 'Important Data';

    return <ChildComponent data={data} />;
};

// ChildComponent
const ChildComponent = ({ data }) => {
    return <GrandchildComponent data={data} />;
};

// GrandchildComponent
const GrandchildComponent = ({ data }) => {
    return <GreatGrandchildComponent data={data} />;
};

// GreatGrandchildComponent
const GreatGrandchildComponent = ({ data }) => {
    return <div>{data}</div>;
};

export default ParentComponent;

```

## Setting Up Context 

```javascript
import React, { createContext } from 'react';

const MyContext = createContext();

function App() {
  return (
    <MyContext.Provider value={{ message: 'Hello from context!' }}>
      {/* Rest of the app */}
    </MyContext.Provider>
  );
}
```

## Using useContext Hook 

```javascript
import React, { useContext } from 'react';
import MyContext from './MyContext';

function MyComponent() {
  const { message } = useContext(MyContext);
  return <div>{message}</div>;
}
```

## Practical Example 

```javascript
export const ThemeContext = createContext({
  theme: "light",
  setTheme: () => {},
});

function SomeComponent() {
  const [theme, setTheme] = useState("light");

    const lightStyle = {
      // styles ...
    };
    const darkStyle = {
      // styles ...
    };

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <div style={theme === "light" ? lightStyle : darkStyle}>
        <ThemeConsumer />
      </div>
    </ThemeContext.Provider>
  );
}

function ThemeConsumer() {
  const { theme, setTheme } = useContext(ThemeContext);

  return (
    <div>
      Current theme: {theme}
      <button onClick={() => setTheme(theme === "light" ? "dark" : "light")}>
        Toggle Theme
      </button>
    </div>
  );
}
```

## Advance uses

### Before.

```jsx
import { useReducer, useState } from "react";

function AddQuest({ onAddQuest }) {
  const [text, setText] = useState("");
  return (
    <>
      <input
        placeholder="Add quest"
        value={text}
        onChange={(e) => setText(e.target.value)}
      />
      <button
        onClick={() => {
          setText("");
          onAddQuest(text);
        }}
      >
        Add
      </button>
    </>
  );
}

function QuestLog({ quests, onChangeQuest, onDeleteQuest }) {
  return (
    <ul>
      {quests.map((quest) => (
        <li key={quest.id}>
          <QuestItem
            quest={quest}
            onChange={onChangeQuest}
            onDelete={onDeleteQuest}
          />
        </li>
      ))}
    </ul>
  );
}

function QuestItem({ quest, onChange, onDelete }) {
  const [isEditing, setIsEditing] = useState(false);
  let questContent;
  if (isEditing) {
    questContent = (
      <>
        <input
          value={quest.text}
          onChange={(e) => {
            onChange({
              ...quest,
              text: e.target.value,
            });
          }}
        />
        <button onClick={() => setIsEditing(false)}>Save</button>
      </>
    );
  } else {
    questContent = (
      <>
        {quest.text}
        <button onClick={() => setIsEditing(true)}>Edit</button>
      </>
    );
  }
  return (
    <label>
      <input
        type="checkbox"
        checked={quest.done}
        onChange={(e) => {
          onChange({
            ...quest,
            done: e.target.checked,
          });
        }}
      />
      {questContent}
      <button onClick={() => onDelete(quest.id)}>Delete</button>
    </label>
  );
}

export default function App() {
  const [quests, dispatch] = useReducer(questsReducer, initialQuests);

  function handleAddQuest(text) {
    dispatch({
      type: "added",
      id: nextId++,
      text: text,
    });
  }

  function handleChangeQuest(quest) {
    dispatch({
      type: "changed",
      quest: quest,
    });
  }

  function handleDeleteQuest(questId) {
    dispatch({
      type: "deleted",
      id: questId,
    });
  }

  return (
    <>
      <h1>Link's Quest Log</h1>
      <AddQuest onAddQuest={handleAddQuest} />
      <QuestLog
        quests={quests}
        onChangeQuest={handleChangeQuest}
        onDeleteQuest={handleDeleteQuest}
      />
    </>
  );
}

function questsReducer(quests, action) {
  switch (action.type) {
    case "added": {
      return [
        ...quests,
        {
          id: action.id,
          text: action.text,
          done: false,
        },
      ];
    }
    case "changed": {
      return quests.map((t) => {
        if (t.id === action.quest.id) {
          return action.quest;
        } else {
          return t;
        }
      });
    }
    case "deleted": {
      return quests.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error("Unknown action: " + action.type);
    }
  }
}

let nextId = 3;

const initialQuests = [
  { id: 0, text: "Find the Master Sword", done: false },
  { id: 1, text: "Complete a Dungeon", done: false },
  { id: 2, text: "Gather 3 Triforce Pieces", done: false },
];


```

Now lets clean up the prop passing with context

First we import createContext and useContext and then initialize them

```jsx


import { useReducer, useState, createContext, useContext } from "react";

const QuestsContext = createContext(null);
const QuestsDispatchContext = createContext(null);


```

Then import and provide them to your parent component

```jsx
import { QuestsContext, QuestsDispatchContext } from './TasksContext.js';

export default function App() {
  const [quests, dispatch] = useReducer(tasksReducer, initialTasks);
  //  ... 
  return (
    <QuestsContext.Provider value={quests}>
      <QuestsDispatchContext.Provider value={dispatch}>
        <h1>Link's Quest Log</h1>
        <AddQuest onAddQuest={handleAddQuest} />
        <QuestLog
          quests={quests}
          onChangeQuest={handleChangeQuest}
          onDeleteQuest={handleDeleteQuest}
        />
      </QuestsDispatchContext.Provider>
    </QuestsContext.Provider>
  );
}
```

Now we dont need to pass props so lets remove those and pull in quests from QuestContext

```jsx
function QuestLog() {
  const { quests } = useContext(QuestsContext);
  return (
    <ul>
      {quests.map((quest) => (
        <li key={quest.id}>
          <QuestItem quest={quest} />
        </li>
      ))}
    </ul>
  );
}

``` 

And for the event handlers we can read QuestsDispatchContext

```jsx
function QuestItem({ quest }) {
  const dispatch = useContext(QuestsDispatchContext);
  // ...
  return (
    <label>
      <input
        type="checkbox"
        checked={quest.done}
        onChange={(e) => {
          dispatch({
            type: "changed",
            quest: {
              ...quest,
              done: e.target.checked,
            },
          });
        }}
      />
      {questContent}
      <button
        onClick={() => {
          dispatch({
            type: "deleted",
            id: quest.id,
          });
        }}
      >
        Delete
      </button>
    </label>
  );
}
```

Now since we have more than one provider we can see a bit of nesting happening. 

```jsx
const App = () => {
  const [globalSettings, setGlobalSettings] = useState({ theme: 'dark', language: 'en' });

  return (
    <GlobalSettingsContext.Provider value={{ globalSettings, setGlobalSettings }}>
      <ResourceIntensiveWidget />
      <ComplexDataVisualizer />
      <HeavyComputationPanel />
    </GlobalSettingsContext.Provider>
  );
};
```

A good way to clean this up is with the AppProviders pattern. 

So lets move all of our providers to the AppProviders components and import that as a single provider.

```jsx
export const AppProvider = ({ children }) => {
  const [nextId, setNextId] = useState(3);

  const questsReducer = (quests, action) => {
    switch (action.type) {
      case "added":
        return [
          ...quests,
          {
            id: action.id,
            text: action.text,
            done: false,
          },
        ];
      case "changed":
        return quests.map((quest) => {
          if (quest.id === action.quest.id) {
            return action.quest;
          } else {
            return quest;
          }
        });
      case "deleted":
        return quests.filter((quest) => quest.id !== action.id);
      default:
        throw new Error("Unknown action: " + action.type);
    }
  };

  const initialQuests = [
    { id: 0, text: "Find the Master Sword", done: false },
    { id: 1, text: "Complete a Dungeon", done: false },
    { id: 2, text: "Gather 3 Triforce Pieces", done: false },
  ];

  const [quests, dispatch] = useReducer(questsReducer, initialQuests);

  const contextValue = {
    quests,
    dispatch,
    getNextId: () => {
      const id = nextId;
      setNextId(nextId + 1);
      return id;
    },
  };

  return (
    <QuestsContext.Provider value={contextValue}>
      <QuestsDispatchContext.Provider value={dispatch}>
        {children}
      </QuestsDispatchContext.Provider>
    </QuestsContext.Provider>
  );
};

```

Now we can import AppProvider

```jsx
import { AppProvider } from './AppProvider.js';

// ...

export default function App() {
  return (
    <AppProvider>
      <h1>Link's Quest Log</h1>
      <AddQuest />
      <QuestLog />
    </AppProvider>
  );
}
```









