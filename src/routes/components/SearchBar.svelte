<script>
    import { createEventDispatcher } from "svelte";
  
    const dispatch = createEventDispatcher();
  
    export let score;
    export let genre;
    export let tags;
    
    const debounce = (func, delay) => {
      let timer;
      return (...args) => {
        clearTimeout(timer);
        timer = setTimeout(() => {
          func(...args);
        }, delay);
      };
    };
  
    const handleInput = debounce((event) => {
      const value = event.target.value;
      score = value;
      genre = value;
      tags = value;
      dispatch("scoreChange", { score });
      dispatch("genreChange", { genre });
      dispatch("tagsChange", { tags });
    }, 750);
  </script>
  
  <form>
    <label for="score-input">Search:</label>
    <input
      type="text"
      class="border w-full rounded-lg focus:outline-none hover:cursor-pointer"
      on:input={handleInput}
    />
  </form>
  