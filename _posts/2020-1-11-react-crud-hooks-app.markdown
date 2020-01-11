---
layout: post
title:  "Creating a CRUD app in React with Hooks"
date:   2020-1-11 10:35:58 +0100
categories: react
---

In this tutorial we will build a create, read, update and delete web application with React using React Hooks. Hooks let us use state and other features in functional components instead of writing class components.

<a href="https://master.d3nd8fc088qo5a.amplifyapp.com/">View demo</a><br/>
<a href="https://github.com/sanderdebr/react-crud-hooks/">View code</a>

This tutorial is divided up in the following sections:

<ol>
    <li>Setting up the project</li>
    <li>Adding users table</li>
    <li>Adding a user</li>
    <li>Deleting a user</li>
    <li>Updating a user</li>
    <li>Using the Effect Hook</li>
</ol>

# 1. Setting up the project

We will start by creating a react app with npm:

{% highlight js %}
npx create-react-app react-crud-hooks
{% endhighlight %}

Then browse to this folder and delete everything from the /src folder except App.js, index.js and index.css

For index.css we will use a simple CSS boilerplate called Skeleton which you can find here:
http://getskeleton.com/

Add the styles in the /public folder into index.html:

{% highlight js %}
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/skeleton/2.0.4/skeleton.min.css">
{% endhighlight %}

Then convert App.js to a functional component and add the following set-up. Notice how easy the skeleton CSS boiler plate works:

{% highlight js %}
import React from 'react'

const App = () => {

  return (
    <div className="container">
      <h1>React CRUD App with Hooks</h1>
      <div className="row">
        <div className="five columns">
          <h2>Add user</h2>
        </div>
        <div className="seven columns">
          <h2>View users</h2>
        </div>
      </div>
    </div>
  )
}

export default App
{% endhighlight %}

# 2. Adding users table

We will retrieve our user data from a separate file. Let’s create data.js inside /src and add an array called users with a couple user object inside and then export it:

{% highlight js %}
const userList = [
    {
        id: 1,
        name: 'Frank',
        username: 'Frank Degrassi'
    },
    {
        id: 2,
        name: 'Birgit',
        username: 'Birgit Boswald'
    }
];

export default userList;
{% endhighlight %}

Then create a folder called /tables and add a file UserTable.jsx. Here we will add a basic table which loops over the users. Notice we are using a ternary operator which is the same as an if/else statement which returns immediately. Also we are destructuring off the object properties so we do not have to rewrite the property again. If there are no users found, we will show an empty cell with some text.

{% highlight js %}
import React from 'react';

const UserTable = (props) => {
    return (
        <table>
            <thead>
                <tr>
                    <th>ID</th>
                    <th>Name</th>
                    <th>Username</th>
                    <th>Actions</th>
                </tr>
            </thead>
            <tbody>
                { props.users.length > 0 ? (
                    props.users.map(user => {
                        const {id, name, username} = user;
                        return (
                            <tr>
                                <td>{id}</td>
                                <td>{name}</td>
                                <td>{username}</td>
                                <td>
                                    <button>Delete</button>
                                    <button>Edit</button>
                                </td>
                            </tr>
                        )
                    })
                ) : (
                    <tr>
                        <td colSpan={4}>No users found</td>
                    </tr>
                )   
                }
            </tbody>
        </table>
    )
}

export default UserTable;
{% endhighlight %}

The table loops over the users received by App.js through the user props. Let’s add them into App.js and also the functionality to retrieve users from data.js which we will do with useState. Every useState has a getter and a setter.

{% highlight js %}
import React, {useState} from 'react'
import userList from './data.js';
import UserTable from './tables/UserTable';

const App = () => {

  const [users, setUsers] = useState(userList);

  return (
    <div className="container">
      <h1>React CRUD App with Hooks</h1>
      <div className="row">
        <div className="six columns">
          <h2>Add user</h2>
        </div>
        <div className="six columns">
          <h2>View users</h2>
          <UserTable users={users} />
        </div>
      </div>
    </div>
  )
}

export default App
{% endhighlight %}

Make sure to import the UserTable in App.js and add the users as props into UserTable.

# 3. Adding a user

Next up we will add the functionality to add a user, first by adding the function into App.js which receives the new user from the Add User component which we will create.

The addUser function puts an object containing a new user into our users array of user objects. We do this by using our setUsers from useState function. By using the spread operator we keep the current user array the same. The ID we will just set based on the current amount of users plus one.

{% highlight js %}
const addUser = user => {
    user.id = users.length + 1;
    setUsers([...users, user]);
  }
{% endhighlight %}

Then we will pass this function to our Add User component:

{% highlight js %}
<AddUserForm addUser={addUser} />
{% endhighlight %}

Which we will create now! Create a folder /forms with a file called AddUserForm.jsx. 

{% highlight js %}
import React, {useState} from 'react';

const AddUserForm = (props) => {

    const initUser = {id: null, name: '', username: ''};

    const [user, setUser] = useState(initUser);

    return (
        <form>
            <label>Name</label>
            <input className="u-full-width" type="text" name=”name” value={user.name} />
            <label>Username</label>
            <input className="u-full-width" type="text" name=”username” value={user.username} />
            <button className="button-primary" type="submit">Add user</button>
        </form>
    )
}

export default AddUserForm;
{% endhighlight %}

Again we are using useState to manage the state of our new user. The initial state of the user values are empty. Now we will add the onChange and onSubmit functions:

For handleChange, we destructure off the properties of the event.target object. Then we dynamically set our object keys based on the used input field:

{% highlight js %}
import React, {useState} from 'react';

const AddUserForm = (props) => {

    const initUser = {id: null, name: '', username: ''};

    const [user, setUser] = useState(initUser);

    const handleChange = e => {
        const {name, value} = e.target;
        setUser({...user, [name]: value});
    }

    const handleSubmit = e => {
        e.preventDefault();
        if (user.name && user.username) props.addUser(user);
    }

    return (
        <form>
            <label>Name</label>
            <input className="u-full-width" type="text" value={user.name} name="name" onChange={handleChange} />
            <label>Username</label>
            <input className="u-full-width" type="text" value={user.username} name="username" onChange={handleChange} />
            <button className="button-primary" type="submit" onClick={handleSubmit} >Add user</button>
        </form>
    )
}

export default AddUserForm;
{% endhighlight %}

Great! Now we can add a user. Notice in our handleSubmit we are preventing the default page refresh and also checking if our user.name and user.username actually have both been filled in.

# 4. Deleting a user

Now we will add the functionality to delete a user, which is quite simple. We will just filter over our users array and filter out the user which has the ID of the user we want to delete. Again we will use our setUsers function to update the new users state.

{% highlight js %}
UserTable.jsx
<button onClick={() => props.deleteUser(id)}>Delete</button>

App.js
const deleteUser = id => setUsers(users.filter(user => user.id !== id));

<UserTable users={users} deleteUser={deleteUser} />
{% endhighlight %}

# 5. Updating a user

Updating a user is a bit more difficult than adding or deleting a user. First we will set up out form in ./forms/EditUserForm.jsx and import it into App.js. We will just copy our AddUserForm.jsx and change the currentUser to the user we are receiving from App.js:

{% highlight js %}
import React, {useState} from 'react';

const EditUserForm = (props) => {

    const [user, setUser] = useState(props.currentUser);

    const handleChange = e => {
        const {name, value} = e.target;
        setUser({...user, [name]: value});
    }

    const handleSubmit = e => {
        e.preventDefault();
        if (user.name && user.username) props.updateUser(user);
    }

    return (
        <form>
            <label>Name</label>
            <input className="u-full-width" type="text" value={user.name} name="name" onChange={handleChange} />
            <label>Username</label>
            <input className="u-full-width" type="text" value={user.username} name="username" onChange={handleChange} />
            <button className="button-primary" type="submit" onClick={handleSubmit} >Edit user</button>
            <button type="submit" onClick={() => props.setEditing(false)} >Cancel</button>
        </form>
    )
}

export default EditUserForm;
{% endhighlight %}

onSubmit we send the updated users back to App.js

In App.js we will use the useState function again to check if the user is currently editing and to decide which user is currently being edited:

{% highlight js %}
const [editing, setEditing] = useState(false);const [editing, setEditing] = useState(false);

const initialUser = {id: null, name: '', username: ''};

const [currentUser, setCurrentUser] = useState(initialUser);
{% endhighlight %}

We will show the AddUser or EditUser form based on editing state:

{% highlight js %}
<div className="container">
      <h1>React CRUD App with Hooks</h1>
      <div className="row">
        <div className="five columns">
          { editing ? (
            <div>
              <h2>Edit user</h2>
              <EditUserForm 
                currentUser={currentUser}
                setEditing={setEditing}
                updateUser={updateUser}
              />
            </div>
          ) : (
            <div>
              <h2>Add user</h2>
              <AddUserForm addUser={addUser} />
            </div>
          )}
        </div>
        <div className="seven columns">
          <h2>View users</h2>
          <UserTable users={users} deleteUser={deleteUser} editUser={editUser} />
        </div>
      </div>
    </div>
{% endhighlight %}

Then we will add our editUser and updateUser functions in App.js:

{% highlight js %}
const editUser = (id, user) => {
  setEditing(true);
  setCurrentUser(user);
}

  const updateUser = (newUser) => {
  setUsers(users.map(user => (user.id === currentUser.id ? newUser : user)))
}
{% endhighlight %}

Great! Now we can edit our users. Let’s fix the last issue in the next section.

# 6. Using the Effect Hook

It is currently not possible to change user while editing, we can fix this by using the effect hook. This is similar to componentDidMount() in class components. First make sure to import useEffect in EditUserForm.jsx

{% highlight js %}
useEffect(() => {
    setUser(props.currentUser)
}, [props])
{% endhighlight %}

This will make when the component re renders, the props are also updated.

Super! We have finished building our React CRUD app with Hooks.

