<script>
  export let title = "Apps";
</script>

<style>
  header {
    width: 100%;
    padding: 10px 20px;
    background-color: rgba(255, 255, 255, 0.1);
    color: black;
    display: flex;
    justify-content: space-between;
    align-items: center;
    transition: transform 0.3s, background-color 0.3s;
  }
  header:hover {
    background-color: rgba(255, 255, 255, 0.2); 
  }
  h1 {
    margin: 0;
    font-size: 1.5rem;
  }
  nav {
    display: flex;
    gap: 20px;
  }
  nav a {
    color: black;
    text-decoration: none;
    font-size: 1rem;
    transition: color 0.3s, transform 0.3s;
  }
  nav a:hover {
    color: #007BFF;
    transform: scale(1.1); 
  }
</style>

<header>
  <h1>{title}</h1>
  <nav>
    <a href="#home">Home</a>
    <a href="#about">About</a>
    <a href="#contact">Contact</a>
  </nav>
</header>
