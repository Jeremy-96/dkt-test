<script>
  /**
	 * Import useful packages from 'svelte-routing'
   * Export some useful variables that will be used in other components
   * Initialize variables used in this component
	 */
  import { link } from 'svelte-routing';
  export let sports = [];


  const paginationLimit = 12;
  let pageCount = Math.ceil(sports.length / paginationLimit);
  let currentPage = 1;
  localStorage.setItem('currentPage', currentPage);
  let searchTag = localStorage.getItem('input');
  let hasDktId = localStorage.getItem('hasDktID');
  let url = window.location.href;
  let searchParams = new window.URL(url).searchParams;
  let page = searchParams.get('page');

  // If the number of the page changes in the url --> update the currentPage variable
  if(page && searchTag.length == 0){
    localStorage.setItem('currentPage', page);
    currentPage = parseInt(page);
  }else {
    currentPage = parseInt(localStorage.getItem('currentPage'));
  }

  /**
   * Go to previous page
   * @function previousPage
   * @author Jérémy Thonon <jeremythonon96@hotmail.com>
   */
  function previousPage(){
    if (currentPage > 1) {
      currentPage -= 1;
      localStorage.setItem('currentPage', currentPage);
      window.history.pushState({}, '', `?page=${currentPage}`);
    }
  }

  /**
   * Go to next page
   * @function nextPage
   * @author Jérémy Thonon <jeremythonon96@hotmail.com>
   */
  function nextPage(){
    if (currentPage < pageCount) {
      currentPage += 1;
      localStorage.setItem('currentPage', currentPage);
      window.history.pushState({}, '', `?page=${currentPage}`);
    }
  }

  /**
   * Return the list of all sports considering the input values given in the search-bar
   * @function searchResult
   * @author Jérémy Thonon <jeremythonon96@hotmail.com>
   */
  function searchResult(){
    let hasId;

    // Check if the checkbox value for decathlon_id is set to true or false and filter the sports based on that
    if (hasDktId == 'true'){
      hasId = sports.filter(sport => sport.attributes.decathlon_id != null);
    } else{
      hasId = sports;
    }

    // Check if the search-bar input value (sport tag) is empty or not and filter the sports based on that
    // Note: the input value is cleaned (string to lowercase + remove white spaces and ponctuation)
    if (searchTag != null && searchTag.length > 0){
      let cleanSearchTag = searchTag.toLowerCase().replace(/[.,\/#!$%\^&\*;:{}=\-_`~() ]/g,"");
      const results = hasId.filter(sport => sport.relationships.tags.data.includes(cleanSearchTag));

      // If the input value gives some results --> CurrentPage is set to 1
      if(results.length > 0){
        localStorage.setItem('currentPage', 1);
      }
      // If the input value gives no result --> CurrentPage is set to 0
      else {
        localStorage.setItem('currentPage', 0);
        currentPage = localStorage.getItem('currentPage');
      }
      
      pageCount = Math.ceil(results.length / paginationLimit)
      return results;

    } else {
      pageCount = Math.ceil(hasId.length / paginationLimit)
      return hasId;
    }
  }
</script>

<!-- HTML start -->
<main>
  <section>
    {#each searchResult() as sport, index}
      {#if (index <= (currentPage * paginationLimit) - 1) && (index >= (currentPage - 1) * paginationLimit)}
        <figure class="sport-card">
          {#if sport.relationships.images.data.length > 0}
            <img class="card-img" src={sport.relationships.images.data[0].url} alt={sport.attributes.name}>
          {/if}

          <figcaption class="card-infos">
            <h3 class="infos-title">{sport.attributes.name}</h3>
            
            {#if sport.attributes.description}
              <p class="infos-description">{sport.attributes.description}</p>
            {/if}

            {#if sport.relationships.tags.data}
            <div class="infos-tags">
              {#each sport.relationships.tags.data.slice(0, 4) as tag, index}
                {#if index < 3}
                  <span class="tag">{tag}</span>
                {:else}
                  <span class="tag">{sport.relationships.tags.data.length - 3} other</span>
                {/if}
              {/each}
            </div>
            {/if}
            <a href={`/sport/${sport.id}`} class="infos-link"  use:link>See more</a>
          </figcaption>
        </figure>
      {/if}
    {/each}

    {#if currentPage == 0}
      <div class="no-results">
        <h2>No results...</h2>
      </div>
    {/if}
  </section>

  <div class="page">
    {#if currentPage > (1)}
      <button on:click={previousPage}  class="previous page-btn" aria-label="Previous">
        <svg width="14" height="21" viewBox="0 0 14 21" fill="none" xmlns="http://www.w3.org/2000/svg">
          <path d="M4.72701 9.95897L12.8586 17.8852L10.4555 20.064L0.00111645 9.87365L10.8166 0.0672866L13.1394 2.33146L4.72701 9.95897Z" fill="white"/>
        </svg>  
        <p>Previous</p>
      </button>
    {:else}
      <button on:click={previousPage}  class="previous page-btn" aria-label="Previous" style="background-color:grey"disabled>
        <svg width="14" height="21" viewBox="0 0 14 21" fill="none" xmlns="http://www.w3.org/2000/svg">
          <path d="M4.72701 9.95897L12.8586 17.8852L10.4555 20.064L0.00111645 9.87365L10.8166 0.0672866L13.1394 2.33146L4.72701 9.95897Z" fill="white"/>
        </svg>
        <p>Previous</p>
      </button>
    {/if}

    <span class="page-indicator">{currentPage} of {pageCount}</span>

    {#if currentPage <= (pageCount - 1) }
      <button on:click={nextPage} class="next page-btn" aria-label="Next">
        <p>Next</p>
        <svg width="13" height="20" viewBox="0 0 13 20" fill="none" xmlns="http://www.w3.org/2000/svg">
          <path d="M8.27334 10L0 2.22187L2.36333 0L13 10L2.36333 20L0 17.7781L8.27334 10Z" fill="white"/>
        </svg>
      </button>
    {:else}
      <button on:click={nextPage} class="next page-btn" aria-label="Next" style="background-color:grey"disabled> 
        <p>Next</p>
        <svg width="13" height="20" viewBox="0 0 13 20" fill="none" xmlns="http://www.w3.org/2000/svg">
          <path d="M8.27334 10L0 2.22187L2.36333 0L13 10L2.36333 20L0 17.7781L8.27334 10Z" fill="white"/>
        </svg>
      </button>
    {/if}
  </div>
</main>
<!-- HTML end -->

<style>
main {
  width: 100%;
  min-height: 500px;
  display: flex;
  flex-direction: column;
  align-items:center;
  justify-content: space-between;
}
section {
  width: 100%;
  height: 80%;
  padding: 2rem;
  display:flex;
  flex-wrap: wrap;
  align-items:center;
  justify-content: center;
}
.sport-card {
  width: 320px;
  height: 380px;
  margin: 1rem;
  display:flex;
  flex-direction: column;
  align-items:center;
  justify-content: space-between;
  box-shadow: 0px 6px 6px rgba(0, 83, 125, 0.1);
  border-radius: 8px 8px 8px 0px;
}
.card-img {
  width: 100%;
  height: 35%;
  border-radius: 8px 8px 0 0;
  object-fit: cover;
}
.card-infos {
  width: 100%;
  height: 65%;
  padding: 0 1.5rem;
  display: flex;
  flex-direction: column;
  align-items:flex-start;
  justify-content: space-around;
}
.infos-title {
  height: 20%;
  font-size: 20px;
  line-height: 24px;
  display:flex;
  align-items: center;
}
.infos-description {
  width: 100%;
  height: 15%;
  text-overflow: ellipsis;
  overflow: hidden;
  white-space: nowrap;
}
.infos-tags {
  height: 20%;
  display: flex;
  align-items: center;
  flex-wrap: wrap;
}
.tag {
  width: auto;
  padding: 2px 8px;
  margin: 4px;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
  background-color: #EFF1F3;
  border-radius: 4px;
}
.infos-link {
  height: 30%;
  margin: 1rem 0 0 0;
  display: flex;
  align-items: center;
  font-size: 18px;
  font-weight: 700;
  line-height: 20px;
  text-decoration: underline;
  color: #00689D;
}
.no-results {
  width: 100%;
  height: 400px;
  display: flex;
  align-items: center;
  justify-content: center;
}
.page {
  width: 100%;
  height: 100px;
  display: flex;
  align-items:center;
  justify-content: center;
}
.page-btn {
  height: 2rem;
  padding: 0.5rem 1rem;
  display: flex;
  justify-content: space-around;
  align-items: center;
  color: #FFFFFF;
  border-radius: 4px;
}
.page-btn svg {
  width:5px;
  height: 8px;
  margin: 0 10px 0 10px;
}
.page-indicator {
  margin: 0 1.5rem;
}

/* Breakpoints for tablet & mobile start */
@media screen and (max-width: 1024px){
  section {
    padding: 2rem 5rem 0 5rem;
    justify-content: space-around;
  }
  .sport-card{
    width: 290px;
    margin: 1rem 0;
  }
  .card-infos {
    padding: 0 1rem;
  }
}

@media screen and (max-width: 768px){
  section {
    padding: 1rem 1rem 0 1rem;
  }
  .sport-card {
    width: 45%;
    height: 225px;
  }
  .card-img {
    height: 50%;
  }
  .infos-title, .infos-link {
    height: 25%;
    padding: 0;
    margin: 0;
    display: flex;
    align-items: center;
    justify-content: center;
  }
  .infos-title {
    font-size: 1rem;
    line-height: 1rem;
    height: 30%;
  }
  .infos-link {
    font-size: 14px;
  }
  .infos-description, .infos-tags{
    display:none;
  }
  .page-btn {
    width:40px;
    height:40px;
    padding: 0;
  }
  .page-btn p{
    display: none;
  }
  .page-btn svg {
    width: 13px;
    height: 20px;
  }
}

@media screen and (max-width: 375px){
  section {
    padding: 1rem 0.5rem 0 0.5rem;
    justify-content: space-around;
  }
  .sport-card {
    height: 150px;
  }
}
/* Breakpoints for tablet & mobile end */
</style>