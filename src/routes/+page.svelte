<script>

    // Set up firebase inmports
    import { initializeApp } from "firebase/app";
    import { getFirestore, collection, onSnapshot, doc, addDoc, updateDoc, deleteDoc } from "firebase/firestore";
    import {firebaseConfig} from "$lib/firebaseConfig";

    // Initialize firebase app using config
    const firebaseApp = initializeApp(firebaseConfig);

    // Get firestore db 
    const db = getFirestore();

    // Create reference to our collection of todos
    const colRef = collection(db, "todos");

    // Declare todos array for storing tasks
    let todos = [];

    // Function that grabs todos from db and adds them to our locally stored object
    const unsubscribe = onSnapshot(colRef, (querySnapshot) => {
        // Declare list of firebase todos
        let fbTodos = [];
        querySnapshot.forEach((doc) => {
            // Create todo object for each doc in the collection
            let todo = {...doc.data(), id: doc.id};
            // Add each object to our list 
            fbTodos = [todo, ...fbTodos];
        });
        todos = fbTodos;
    });

    // Console log to make sure initialization worked correctly
    console.log({firebaseApp, db});

    // Declare task and error variables, default to empty
    let task = "";
    let error = "";

    // Function that adds a todo when add button is pressed
    const addTodo = async () => {
        // If text box is non empty
        if (task !== "") {
            // Add a new task with task name to database
            const docRef = await addDoc(collection(db, "todos"), {
                task: task,
                isComplete: false,
                createdAt: new Date(),
            });
            // Reset error
            error = "";
        // If text box is empty
        } else {
            // Change error message
            error = "Please input task name before adding."
        }
        // Reset task string/input text box after 
        task = "";
    };

    // Update the isComplete field of relevant todo 
    const markTodoAsComplete = async (todo) => {
        await updateDoc(doc(db, "todos", todo.id.toString()), {
            isComplete: !todo.isComplete
        });
    };

    // Delete the todo with the given id from the database
    const deleteTodo = async (id) => {
        await deleteDoc(doc(db, "todos", id.toString()));
    };

    // Add todo when user presses enter key
    const keyIsPressed = (event) => {
        if (event.key === "Enter") {
            addTodo();
        }
    };
</script>

<!-- Header with title and text bar for adding new tasks -->
<div id="headerDiv" class="header">
    <!-- Header with title of the webpage -->
    <h2>My To Do List</h2>
    <!-- Counter with tasks and completed tasks -->
    <p>Number of Tasks: {todos.length}</p>
    <p>Number of Tasks Completed: {todos.filter(d => d.isComplete).length}</p>
    <span>
        <!-- Add text box for typing in task name -->
        <input type="text" placeholder="Add a task" bind:value={task}>
        <!-- Create add button to add new tasks -->
        <button class="add" on:click={addTodo}>Add</button>
    </span>
</div>

<!-- Todo List element is an unordered list with no bullets -->
<ul>
    <!-- Create list item for each task  -->
    {#each todos as todo}
        <!-- Task item, completes the task when clicked -->
        <li class:complete={todo.isComplete} on:click={() => markTodoAsComplete(todo)}>
            <!-- Span with task name -->
            <span>
                {todo.task}
            </span>
            <!-- Delete button that deletes task on click -->
            <span>
                <button class="close" on:click={() => deleteTodo(todo.id)}> ❌ </button>
            </span>
        </li>
        <!-- If list is empty, show no todos found message -->
        {:else }
            <p>No todos found!</p>
    {/each}
    <!-- Placeholder for error if user tries to add empty task -->
    <p class="error">{error}</p>
</ul>

<!-- Check when enter button is pressed to enable adding tasks with enter -->
<svelte:window on:keydown={keyIsPressed}/>

<!-- Css styling -->
<style>
    /* Set color and font for error message. */
    .error {
        color: red;
        font-family: sans-serif;
    }

    /* Set font for text elements (counters, no todos found). */
    p {
        font-family: sans-serif;
    }

    /* Set margins and padding to 0 for the list. */
    ul {
        margin: 0;
        padding: 0;
    }

    /* Set properties for all the list items. */
    ul li {
        cursor: pointer;
        position: relative;
        padding: 12px 8px 12px 40px;
        /* Standard background color (white), font/font size,
            transition time for the list items. */
        background: #eee;
        font-size: 18px;
        transition: 0.2s;
        font-family: sans-serif;

        /* Make sure the list items cannot be selected. */
        -webkit-user-select: none;
        -moz-user-select: none;
        -ms-user-select: none;
        user-select: none;
    }

    /* Set the odd numbered list items to a slightly darker color to create separation. */
    ul li:nth-child(odd) {
        background: #f9f9f9;
    }

    /* Darken background color slightly when a user hovers on a task. */
    ul li:hover {
        background: #ddd;
    }

    /* Darken the background and strike through the text when the item is checked. */
    ul li.complete {
        background: #888;
        color: #fff;
        text-decoration: line-through;
    }

    /* Set the properties for the input text box. */
    input {
        margin: 0;
        border: none;
        border-radius: 0;
        width: 75%;
        padding: 10px;
        float: left;
        font-size: 16px;
    }

    /* Set the properties for the Add task button. */
    .add {
        padding: 10px;
        width: 20%;
        background: #d9d9d9;
        color: #555;
        float: left;
        text-align: center;
        font-size: 16px;
        cursor: pointer;
        transition: 0.3s;
        border-radius: 0;
        border: none;
    }

    /* Darken the add button on hover to show that it can be clicked. */
    .add:hover {
        background-color: #bbb;
    }

    /* Set the properties for the page header. */
    .header {
        background-color: #deb82f;
        padding: 30px 40px;
        color: black;
        text-align: center;
        font-family: sans-serif;
    }

    /* Insert some empty space at the bottom of the header. */
    .header:after {
        content: "";
        display: table;
        clear: both;
    }

    /* Set the properties for the delete button. */
    .close {
        cursor: pointer;
        position: absolute;
        /* Make it right aligned with a bit of padding. */
        right: 0;
        top: 0;
        bottom: 0;
        padding: 12px 16px 12px 16px;
        border: none;
        background: none;
    }

    /* Make the delete button darker background on hover. */
    .close:hover {
        background-color: #bbb;  
    }
</style>
