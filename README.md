class AddTodo extends Component {
  constructor(props) {
    super(props);
    const { title, completed, createdAt } = this.props.todo;
    this.state = {
      title,
      completed,
      createdAt,
    };
  }

  onSubmit = () => {
    if (this.state.title.length > 0) this.props.onAdd(this.state);
    return null;
  };

  setStateUtil = (property, value = undefined) => {
    this.setState({
      [property]: value,
    });
  };

  render() {
    const { title, completed } = this.state;
    const { onBlur } = this.props;
    return (
      <View
        style={{
          flex: 1,
          width: '100%',
          flexDirection: 'row',
          alignItems: 'center',
          paddingRight: 10,
          paddingBottom: 5,
          paddingTop: 5,
        }}
      >
        <CheckBox checked={completed} onPress={() => this.setStateUtil('completed', !completed)} />
        <Body
          style={{
            flex: 1,
            justifyContent: 'flex-start',
            alignItems: 'flex-start',
            paddingLeft: 25,
          }}
        >
          <TextInput
            style={{ width: '90%' }}
            placeholder="What needs to be done?"
            autoFocus
            underLineColorAndroid="transparent"
            underlineColor="transparent"
            blurOnSubmit
            onSubmitEditing={this.onSubmit}
            onChangeText={changedTitle => this.setStateUtil('title', changedTitle)}
            value={title}
            autoCorrect={false}
            autoCapitalize="none"
            onBlur={onBlur}
          />
        </Body>
        <TouchableOpacity
          onPress={() => this.props.onCancelDelete}
          style={{ paddingLeft: 25, paddingRight: 15 }}
        >
          <Ionicons
            name="ios-trash-outline"
            color={`${title.length > 0 ? 'black' : 'grey'}`}
            size={23}
          />
        </TouchableOpacity>
      </View>
    );
  }
}
...
import AddTodoButton from './AddTodoButton';
import AddTodo from './AddTodo';
import TodoModel from '../api/todos';
...


export default class TodosContainer extends React.Component {
  state = {
    addingTodo: false,
  };

  componentDidMount() {
    
    this.api = new TodoModel('react-todos');
  }

  render() {
    return (
      <View style={styles.container}>
        <Header />
        <StatusBar backgroundColor={COLORS.primary} barStyle="light-content" />
        <ScrollView>
          {this.state.addingTodo ? (
            <View style={styles.row}>
              <AddTodo
                onAdd={(todo) => {
                  this.setState({ addingTodo: false });
                  this.api.add(todo);
                }}
                onCancelDelete={() => this.setState({ addingTodo: false })}
                onBlur={() => this.setState({ addingTodo: false })}
              />
            </View>
          ) : null}
        </ScrollView>
        <AddTodoButton onPress={() => this.setState({ addingTodo: true })} />
      </View>
    );
  }
}

import { ReactiveList } from '@appbaseio/reactivesearch-native';
...

export default class TodosContainer extends React.Component {
  render() {
    return (
      <View style={styles.container}>
        <Header />
        <StatusBar backgroundColor={COLORS.primary} barStyle="light-content" />
        <ScrollView>
          <ReactiveList
            componentId="ReactiveList"
            defaultQuery={() => ({
              query: {
                match_all: {},
              },
            })}
            stream
            onAllData={this.onAllData}
            dataField="title"
            showResultStats={false}
            pagination={false}
          />
          ...
        </ScrollView>
        <AddTodoButton onPress={() => this.setState({ addingTodo: true })} />
      </View>
    );
  }
}

...
import { ScrollView, StyleSheet, StatusBar, FlatList, Text } from 'react-native';
import CONSTANTS from '../constants';
...

export default class TodosContainer extends React.Component {
  ...
  onAllData = (todos, streamData) => {
    

    return (
      <FlatList
        style={{ width: '100%', top: 15 }}
        data={filteredData}
        keyExtractor={item => item._id}
        renderItem={({ item: todo }) => (
            <Text>{todo.title}</Text>
        )}
      />
    );
  };

  filterTodosData = (todosData) => {
    const { screen } = this.props;

    switch (screen) {
      case CONSTANTS.ALL:
        return todosData;
      case CONSTANTS.ACTIVE:
        return todosData.filter(todo => !todo.completed);
      case CONSTANTS.COMPLETED:
        return todosData.filter(todo => todo.completed);
    }

    return todosData;
  };

  render() {
    ...
  }
}

class TodoItem extends Component {
  onTodoItemToggle = (todo, propAction) => {
    propAction({
      ...todo,
      completed: !todo.completed,
    });
  };

  render() {
    const { todo, onUpdate, onDelete } = this.props;

    return (
      <View style={styles.row}>
        <View
          style={{
            flex: 1,
            width: '100%',
            flexDirection: 'row',
            alignItems: 'center',
            paddingRight: 10,
            paddingVertical: 5,
          }}
        >
          <TouchableOpacity
            onPress={() => this.onTodoItemToggle(todo, onUpdate)}
            style={{
              flex: 1,
              width: '100%',
              flexDirection: 'row',
            }}
          >
            <CheckBox
              checked={todo.completed}
              onPress={() => this.onTodoItemToggle(todo, onUpdate)}
            />
            <Body
              style={{
                flex: 1,
                justifyContent: 'flex-start',
                alignItems: 'flex-start',
                paddingLeft: 25,
              }}
            >
              <Text
                style={{
                  color: todo.completed ? 'grey' : 'black',
                  textDecorationLine: todo.completed ? 'line-through' : 'none',
                }}
              >
                {todo.title}
              </Text>
            </Body>
          </TouchableOpacity>
          <TouchableOpacity
            onPress={() => onDelete(todo)}
            style={{ paddingLeft: 25, paddingRight: 15 }}
          >
            <Ionicons
              name="ios-trash-outline"
              color={`${todo.title.length > 0 ? 'black' : 'grey'}`}
              size={23}
            />
          </TouchableOpacity>
        </View>
      </View>
    );
  }
}

class TodosContainer extends Component {
  ...
  onAllData = (todos, streamData) => {
    ...
    return (
      <FlatList
        ...
        renderItem={({ item: todo }) => (
          <TodoItem 
            todo={todo}
            onUpdate={this.api.update} 
            onDelete={this.api.destroy}
          />
        )}
      />
    );
  }
}
import Utils from '../utils';
...

export default class TodosContainer extends React.Component {
  ...
  onAllData = (todos, streamData) => {
    
    const filteredData = this.filterTodosData(todosData);

    return (
      <FlatList
        style={{ width: '100%', top: 15 }}
        data={filteredData}
        keyExtractor={item => item._id}
        renderItem={({ item: todo }) => (
            <TodoItem todo={todo} onUpdate={this.api.update} onDelete={this.api.destroy} />
        )}
      />
    );
  };
  ...
}


class Utils {
  static mergeTodos(todos, streamData) {
    // generate an array of ids of streamData
    const streamDataIds = streamData.map(todo => todo._id);

    return (
      todos
        // consider streamData as the source of truth
        // first take existing todos which are not present in stream data
        .filter(({ _id }) => !streamDataIds.includes(_id))
        // then add todos from stream data
        .concat(streamData)
        // remove todos which are deleted in stream data
        .filter(todo => !todo._deleted)
        // finally sort on the basis of creation timestamp
        .sort((a, b) => a.createdAt - b.createdAt)
    );
  }
}

