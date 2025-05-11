<script>
export default {
  name: "BookView",

  data() {
    return {
      books: [] // An empty array to hold book data
    };
  },
  
  mounted() {
    console.log("BookView component has been loaded.");
    this.fetchBooks(); // Fetch books when the component is mounted
  },


  //region Methods
  methods: {
    SubmitInput() {
      var titleInput = document.getElementById("titleInput");
      var authorInput = document.getElementById("authorInput");
      var pagesInput = document.getElementById("pagesInput");
      this.handleSubmit("/create/book")

    },
  cancelInput(){
      var titleInput = document.getElementById("titleInput");
      var authorInput = document.getElementById("AuthorInput");
      var pagesInput = document.getElementById("PagesInput");

      titleInput.value = "";
      authorInput.value = "";
      pagesInput.value = "";
  },
  UpdateInput() {
      this.deleteInput();
      this.SubmitInput();
      
  },

  deleteInput() {
      var titleInput = document.getElementById("titleInput");
      var authorInput = document.getElementById("AuthorInput");
      var pagesInput = document.getElementById("PagesInput");
      const bookSelect = document.getElementById("bookSelect");
      // get the ID of the book 
      const id = parseInt(bookSelect.value);
      // call handlesubmit with the delete route
      const myHeaders = new Headers();
      myHeaders.append("Origin", "http://localhost:5173");

      const requestOptions = {
        method: "DELETE",
        headers: myHeaders,
        redirect: "follow"
      };

      fetch(`http://localhost:8080/delete/book/${id}`, requestOptions)
        .then((response) => response.text())
        .then((result) => console.log(result))
        .catch((error) => console.error(error));
        },

     
  async handleSubmit(route, update = false)  // update is a boolean to check if we are updating or creating a book
  {
    const endpoint = "http://localhost:8080";
    const title = document.querySelector(".titleInput").value;
    const author = document.querySelector(".AuthorInput").value;
    const pages = parseInt(document.querySelector(".PagesInput").value);
    let id = 1;
  

    // lazy ID finder of incremental ID because we're not using a true database 
    // correct way of loading all books 
    // update mode if its false don't overwrite the ID
    if (!update) {
      const books = await this.fetchBooks();
      books.forEach((book) => {
        if (book.id === id) {
          id++;
        }
        });
      }
      // otherwise we update the book 
      else {
        // if we are updating we need to get the ID from the select box
        const bookSelect = document.getElementById("bookSelect");
        // get the ID of the book 
        id = parseInt(bookSelect.value);
      }
      


    const queryParams = new URLSearchParams({
      id: id.toString(),
      title: title,
      author: author,
      pages: pages.toString(),
      });

    const myHeaders = new Headers();
    myHeaders.append("Origin", "http://localhost:5173"); // endpoint for dev 
    myHeaders.append("Content-Type", "application/x-www-form-urlencoded");

    const urlencoded = new URLSearchParams();
    urlencoded.append("id", id);
    urlencoded.append("title", title);
    urlencoded.append("author", author);
    urlencoded.append("pages", pages.toString());

    const requestOptions = {
      method: "POST",
      headers: myHeaders,
      body: urlencoded,
      redirect: "follow"
      };

    fetch(`${endpoint}${route}`, requestOptions)
      .then((response) => response.text())
      .then((result) => alert(result))
      .catch((error) => alert(error));
  },

  async fetchBooks() {
    let id = 1; // stating ID for the first book
    const books = [];
    while (true) {
      try {
        //Failed to execute 'json' on 'Response is what we want to see as an error (its currently not a problem so dont worry)
        const response = await fetch(`http://localhost:8080/get/book/${id}`);
        const data = await response.json();
        if (data && Object.keys(data).length !== 0) {
          console.log("Book ID: ", data.id);
          console.log("Book Title: ", data.title);
          console.log("Book Author: ", data.author);
          console.log("Book Pages: ", data.pages);
          books.push({
            id: data.id,
            title: data.title,
            author: data.author,
            pages: data.pages,
          });
          id++;
        } else {
          break;
        }
      } catch (error) {
        console.error("Error fetching book data:", error);
        break;
      }
    }
    this.books = books; // Update the books array in the component's data
    return books;
  },

  onSelectBook() {
    const bookSelect = document.getElementById("bookSelect");
    const selectedBookId = parseInt(bookSelect.value); // Get the selected book's ID

    if (selectedBookId === 0) {
      // If "Select a book" is chosen, clear the input fields
      document.getElementById("titleInput").value = "";
      document.getElementById("AuthorInput").value = "";
      document.getElementById("PagesInput").value = "";
      return;
    }

    // Find the selected book in the books array
    const selectedBook = this.books.find((book) => book.id === selectedBookId);

    if (selectedBook) {
      // Populate the input fields with the selected book's details
      document.getElementById("titleInput").value = selectedBook.title;
      document.getElementById("AuthorInput").value = selectedBook.author;
      document.getElementById("PagesInput").value = selectedBook.pages;
    }
  },
  refresh(){

    window.location.reload();
  }

  }
};
//endregion methods
</script>

<template>
  <div class="formStyle">
    <select id="bookSelect" @change="onSelectBook">
      <option value="0">Select a book</option>
      <option v-for="book in books" :key="book.id" :value="book.id">
        {{ book.title }} by {{ book.author }}
      </option>
    </select>
    <h1 style="text-align: center; color: aliceblue; font-family:Impact, Haettenschweiler, 'Arial Narrow Bold', sans-serif">Book editing</h1>
    <form >
      <input type="text" class="titleInput inp" id="titleInput" placeholder="book title" />
      <input type="text" class="AuthorInput inp" id="AuthorInput" placeholder="book author" />
      <input type="pages" class="PagesInput inp" id="PagesInput" placeholder="book pages" />
      <br></br>
      <input type="button" class="submitButton butt" id="submitButton" value="Create" @click="SubmitInput" />
      <input type="button" class="deleteButton butt" id="deleteButton" value="Delete" @click="deleteInput" />
      <input type="button" class="updateButton butt" id="updateButton" value="Update" @click="UpdateInput" />
      <input type="button" class="cancelButton butt" id="cancelButton" value="Cancel" @click="cancelInput" />
      <input type="button" class="refresh butt" id="refresh" value="refresh" @click="refresh" />
    </form>
  </div>
</template>

<style>
@media (min-width: 1024px) {
  .about {
    min-height: 100vh;
    display: flex;
    align-items: center;
  }
}


.formStyle{
  padding: 5%;
  background-color: rgb(80, 80, 80);
  flex: 1;
}


.inp{
  padding: 1%;
  margin: 1%;
  border-radius: 5px;
  border: 1px solid black;
  background-color: rgb(80, 80, 80);
  color: white;

}

.butt{
  padding: 1%;
  margin: 1%;
  border-radius: 5px;
  border: 1px solid black;
  background-color: rgb(80, 80, 80);
  color: white;
}

</style>
