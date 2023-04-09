<!-- ExampleComponent.svelte -->
<script>
    import { onMount } from "svelte";
    
    let aniData = [];
    
    const query = `
      query ($page: Int) {
        Page (page: $page, perPage: 10) {
          media {
            id,
            title {
              romaji,
              english,
              native
            },
            episodes,
            genres
          }
        }
      }`;
    
    const variables = { "page": 1 };
    
    const fetchResults = async () => {
      const url = "https://graphql.anilist.co";
      const options = {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          "Accept": "application/json"
        },
        body: JSON.stringify({
          query: query,
          variables: variables
        })
      };
      try {
        const response = await fetch(url, options);
        const data = await response.json();
        aniData = data.data.Page.media;
      } catch (err) {
        console.error(err);
      }
    };
    
    onMount(() => {
      fetchResults();
    });
  </script>
    
  <div>
    <ul>
      {#each aniData as anime}
        <li>
          <h3>{anime.title.romaji == null ? '' : anime.title.romaji}</h3>
          <h3>{anime.title.english == null ? '' : anime.title.english}</h3>
          <h3>{anime.title.native == null ? '' : anime.title.native}</h3>
          <p>Episodes: {anime.episodes}</p>
          <p>Genres: {anime.genres}</p>
        </li>
      {/each}
    </ul>
  </div>
  