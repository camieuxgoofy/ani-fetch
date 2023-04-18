<script>
  import { createEventDispatcher } from "svelte";

  const dispatch = createEventDispatcher();

  export let score;
  export let genre;
  export let tags;
  export let search;

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
    const value = event.target.value.trim();
    let tagArray = [];
    if (typeof value === "string") {
      tagArray = value.split(", ");
    }
    score = value;
    genre = value;
    tags = tagArray;
    search = value;
    dispatch("scoreChange", { score });
    dispatch("genreChange", { genre });
    dispatch("tagsChange", { tags });
    dispatch("searchChange", { search });
  }, 750);
</script>

<input
  type="text"
  class="border w-full text-black rounded-sm focus:outline-none hover:cursor-pointer"
  on:input={handleInput}
/>
