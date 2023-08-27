<script>
      import { db } from "../../lib/firebase/firebase";
      import { authHandlers, authStore } from "../../store/store";
      import { getDoc, doc, setDoc } from "firebase/firestore";
    
      let todoList = [];
      let currTodo = '';
      let error = false;
    
      // Subscribe to the authStore to keep todoList in sync with user data
      authStore.subscribe((curr) => {
        todoList = curr.data.todos;
      });
    
      function addTodo() {
        error = false;
        if (!currTodo) {
          error = true;
          return; // Stop adding if there's no text
        }
        todoList = [...todoList, currTodo];
        currTodo = '';
      }
    
      function editTodo(index) {
        // Replace the todo at the specified index with currTodo
        if (index >= 0 && index < todoList.length) {
          todoList[index] = currTodo;
          currTodo = ''; // Reset currTodo
        }
      }
    
      function deleteTodo(index) {
        // Remove the todo at the specified index
        if (index >= 0 && index < todoList.length) {
          todoList.splice(index, 1);
        }
      }
    
      async function handleLogout() {
        try {
          // Call the logout function from authHandlers (assuming it exists)
          await authHandlers.logout();
        } catch (err) {
          console.error('Error during logout:', err);
        }
      }

      async function saveTodos() {
        try {
            const userRef = doc(db, "users", $authStore.user.uid);
            await setDoc(
                userRef,
                {
                    todos: todoList,
                },
                { merge: true }
            );
        } catch (err) {
            console.log("There was an error saving your information");
        }
    }
    </script>
    
    {#if !$authStore.loading}
    
    <div class="mainContainer">
      <div class="headerContainer">
        <h1>Todo List</h1>
        <div class="headerButtons">
            <button on:click={saveTodos} ><i class="fa-solid fa-bookmark"></i><p>Save</p></button>
             <button on:click={handleLogout} ><i class="fa-solid fa-arrow-right-from-bracket"></i><p>Logout</p></button>
      </div>
      </div>
      <main>
            {#if todoList.length === 0 } 
            <p>
                  You have nothing to do!
            </p>
            {/if}
        {#each todoList as todo, index}
        <div class="todo">
          <p>
            {index + 1}. {todo}
          </p>
          <div class="actions">
            <i
              on:click={() => editTodo(index)}
              on:keydown={() => {}}
              class="fa-solid fa-pen-to-square"
              role="button"
              aria-label="Edit"
              tabindex="0" 
            ></i>
            <i
              on:click={() => deleteTodo(index)}
              on:keydown={() => {}}
              class="fa-solid fa-trash-can"
              role="button"
              aria-label="Delete"
              tabindex="0" 
            ></i>
          </div>
          
          
        </div>
        {/each}
      </main>
      <div class={"enterTodo " + (error ? 'errorBorder' : "")}>
        <input bind:value={currTodo} type="text" placeholder="Enter todo" />
        <button on:click={addTodo}>ADD</button>
      </div>
    </div>
    {/if}

<style>
      .mainContainer {
            display: flex;
            flex-direction: column;
            min-height: 100vh;
            gap: 24px;
            padding: 24px;
            width: 100%;
            max-width: 1000px;
            margin: 0 auto;
      }

      .headerContainer {
            display: flex;
            align-items: center;
            justify-content: space-between;
      }

      .headerButtons {
            display: flex;
            align-items: center;
            gap: 14px;
      }

      .headerContainer button {
            background: white;
            color: blue;
            padding: 10px 18px;
            border: none;
            border-radius: 4px;
            font-weight: 700;
            display: flex;
            align-items: center;
            gap: 10px;
            cursor: pointer;

      }

      .headerContainer button i{
            font-size: 1.1rem;
      }

      .headerContainer button:hover {
            opacity: 0.7;

      }

      main {
            display: flex;
            flex-direction: column;
            gap: 8px;
            flex: 1;
      }

      .todo {
            border-left: 1px solid white;
            padding: 8px 14px;
            display: flex;
            align-items: center;
            justify-content: space-between;
      }

      .actions {
            display: flex;
            align-items: center;
            gap: 14px;
            font-size: 1.3rem;

      }

      .actions i {
            cursor: pointer;
      }

      .actions i:hover {
            color: orange;
      }

      .enterTodo {
            display: flex;
            align-items: stretch;
            border: 1px solid white;
            border-radius: 5px;
            overflow: hidden;
      }

      .errorBorder {
            border-color: orange !important;
      }

      .enterTodo input {
            background: transparent;
            border: none;
            padding: 14px;
            color: white;
            flex: 1;
      }

      .enterTodo input:focus {
            outline: none;
      }

      .enterTodo button {
            padding: 0 28px;
            background: #fff;
            border: none;
            color: blue;
            font-weight: 600;
            cursor: pointer;
      }

      .enterTodo button:hover {
            opacity: 0.7;
      }


</style>