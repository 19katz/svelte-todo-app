<script>
    import { initializeApp } from "firebase/app";
    import { getFirestore, collection, onSnapshot, doc, addDoc, updateDoc, deleteDoc } from "firebase/firestore";
    import { firebaseConfig } from "$lib/firebaseConfig";
    
    // Initialize Firebase and get the corresponding database from Firestore.
    const firebaseApp = initializeApp(firebaseConfig);
    const db = getFirestore();
    // Get the todos collection from the database.
    const colRef = collection(db, "todos");
    console.log(db);

    // Stores todos locally. This is used to update the website, and
    // it updates with the data from Firebase.
    let todos = [
    ];

    // Unsubscribes to the store
    // onSnapshot serves the purpose of being a listener, which will be used
    // to update the data on the website.
    // In this case, we look at each document in the query snapshot, which represents
    // a todo, and add it to a list of Firebase todos (fbTodos).
    // This list of todos is then set to our internal variable for the website,
    // todos, which is used to update what is displayed.
    const unsubscribe = onSnapshot(colRef, (querySnapshot) => {
        let fbTodos = [];
        querySnapshot.forEach((doc) => {
            let todo = {...doc.data(), id: doc.id}
            fbTodos = [...fbTodos, todo];
        });
        console.log(fbTodos);
        todos = fbTodos;
    });

    let task = "";

    // Function to add a new todo to the list by adding it to Firestore.
    // Uses addDoc (which adds todos with a default id to the database)
    // whenever a new todo is made by the user. 
    const addTodo = async () => {
        if (task != "") {
            const docRef = await addDoc(collection(db, "todos"), {
                task: task,
                isComplete: false,
                createdAt: new Date()
            });
        }
        task = "";
    };

    // Uses updateDoc, which updates a field of the database entry/document
    // to update the completion status of a todo by toggling it.
    const markTodoComplete = async (item) => {
        await updateDoc(doc(db, "todos", item.id), {
            isComplete: !item.isComplete
        });
    }

    // Uses deleteDoc, which deletes a document by its id from the database.
    const deleteTodo = async (id) => {
        await deleteDoc(doc(db, "todos", id));
    }

    // When the enter key is pressed, add the todo using the addTodo function
    // defined above.
    const keyIsPressed = (event) => {
        if (event.key == "Enter") {
            addTodo();
        }
    }

    $: console.table(todos);
</script>

<!--Adds html code for the area to submit a new task.-->
<input type="text" placeholder="Add a task" bind:value={task} />
<button on:click={addTodo}>Add</button>

<ol>
    <!--Loop over all the todos in the todo list that has been updated from Firestore.-->
    {#each todos as todo}
        <!--Use the isComplete field of the todo to show its class-->
        <li class:complete={todo.isComplete}>
            <span>
                <!--Display the text of the task-->
                {todo.task}
            </span>
            <span>
                <!--Define buttons for marking a todo as finished and deleting it.
                When the check is clicked, call markTodoComplete. When the x is clicked,
                call deleteTodo.-->
                <button on:click={() => markTodoComplete(todo)}>✔</button>
                <button on:click={() => deleteTodo(todo.id)}>⨉</button>
            </span>
        </li>
    {/each}
</ol>

<!--When a key is held down, call keyIsPressed to check if it's the enter key.-->
<svelte:window on:keydown={keyIsPressed} />

<!--Styling for the "complete" class of each list item.-->
<style>
    .complete {
        text-decoration: line-through;
    }
</style>