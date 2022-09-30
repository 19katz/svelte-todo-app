<script>
    import { initializeApp } from "firebase/app";
    import { getFirestore, collection, onSnapshot, doc, addDoc, updateDoc, deleteDoc } from "firebase/firestore";
    import { firebaseConfig } from "$lib/firebaseConfig";
    
    const firebaseApp = initializeApp(firebaseConfig);
    const db = getFirestore();
    const colRef = collection(db, "todos");
    console.log(db);

    let todos = [
    ];

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

    const markTodoComplete = async (item) => {
        await updateDoc(doc(db, "todos", item.id), {
            isComplete: !item.isComplete
        });
    }

    const deleteTodo = async (id) => {
        await deleteDoc(doc(db, "todos", id));
    }

    const keyIsPressed = (event) => {
        if (event.key == "Enter") {
            addTodo();
        }
    }

    $: console.table(todos);
</script>

<input type="text" placeholder="Add a task" bind:value={task} />
<button on:click={addTodo}>Add</button>

<ol>
    {#each todos as todo}
        <li class:complete={todo.isComplete}>
            <span>
                {todo.task}
            </span>
            <span>
                <button on:click={() => markTodoComplete(todo)}>✔</button>
                <button on:click={() => deleteTodo(todo.id)}>⨉</button>
            </span>
        </li>
    {/each}
</ol>

<svelte:window on:keydown={keyIsPressed} />

<style>
    .complete {
        text-decoration: line-through;
    }
</style>