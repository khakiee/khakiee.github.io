<script>
  import "../app.css";
  import Footer from "../components/footer.svelte";
  import Header from "../components/header.svelte";
</script>

<Header />
<div id="global-layout" class="container min-h-screen">
  <slot/>
</div>
<Footer />