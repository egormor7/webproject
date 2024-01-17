<template>
  <div id="app">
    <Header :user="user"/>
    <Middle :posts="posts" :users="users"/>
    <Footer/>
  </div>
</template>

<script>
import Header from "./components/Header";
import Middle from "./components/Middle";
import Footer from "./components/Footer";
import axios from "axios"

export default {
  name: 'App',
  components: {
    Footer,
    Middle,
    Header
  },
  data: function () {
    return {
      user: null,
      posts: [],
      users: []
    }
  },
  beforeMount() {
    if (localStorage.getItem("jwt") && !this.user) {
      this.$root.$emit("onJwt", localStorage.getItem("jwt"));
    }

    axios.get("/api/1/posts").then(response => {
      this.posts = response.data;
    });
    axios.get("/api/1/users").then(response => {
      this.users = response.data;
    });
  },
  beforeCreate() {
    this.$root.$on("onEnter", (login, password) => {
      if (password === "") {
        this.$root.$emit("onEnterValidationError", "Password is required");
        return;
      }

      axios.post("/api/1/jwt", {
        login, password
      }).then(response => {
        localStorage.setItem("jwt", response.data);
        this.$root.$emit("onJwt", response.data);
      }).catch(error => {
        this.$root.$emit("onEnterValidationError", error.response.data);
      });
    });

    this.$root.$on("onRegister", (login, name, password) => {
      if (password.length < 2 || password.length > 60) {
        this.$root.$emit("onRegisterValidationError", "Password should be at length " +
            "between 2 and 60 characters");
        return;
      }
      if (name.length < 2 || name.length > 24) {
        this.$root.$emit("onRegisterValidationError", "Name should be at length " +
            "between 2 and 24 characters");
        return;
      }
      if (!(/^[a-zA-Z]{2,12} [a-zA-Z]{2,12}$/.test(name))) {
        this.$root.$emit("onRegisterValidationError", "Expected name is two words with latin letters only");
        return;
      }
      if (login.length < 2 || login.length > 24) {
        this.$root.$emit("onRegisterValidationError", "Login should be at length " +
            "between 2 and 24 characters");
        return;
      }
      if (!(/^[a-zA-Z]{2,24}$/.test(login))) {
        this.$root.$emit("onRegisterValidationError", "Expected latin letters only in login");
        return;
      }

      axios.post("/api/1/users", {
        login, name, password
      }).then(response => {
        localStorage.setItem("jwt", response.data);
        this.$root.$emit("onJwt", response.data);
        axios.get("/api/1/users").then(response => {
          this.users = response.data;
        });
      }).catch(error => {
        this.$root.$emit("onRegisterValidationError", error.response.data);
      });
    });

    this.$root.$on("onJwt", (jwt) => {
      localStorage.setItem("jwt", jwt);

      axios.get("/api/1/users/auth", {
        params: {
          jwt
        }
      }).then(response => {
        this.user = response.data;
        this.$root.$emit("onChangePage", "Index");
      }).catch(() => this.$root.$emit("onLogout"))
    });

    this.$root.$on("onLogout", () => {
      localStorage.removeItem("jwt");
      this.user = null;
    });

  }
}
</script>

<style>
#app {

}
</style>
