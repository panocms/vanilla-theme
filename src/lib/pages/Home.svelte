<!-- Post Cards -->
<Posts posts="{data.posts}" />
<!-- Post Cards End -->

<!-- Pagination -->
{#if data.postCount > 0}
  <Pagination
    page="{data.page}"
    totalPage="{data.totalPage}"
    loading="{false}"
    on:firstPageClick="{() => reloadData(1)}"
    on:lastPageClick="{() => reloadData(data.totalPage)}"
    on:pageLinkClick="{(event) => reloadData(event.detail.page)}" />
{/if}

<!-- Pagination End -->
<script context="module">
  import HomeSidebar, {
    load as loadSidebar,
  } from "$lib/component/sidebars/HomeSidebar.svelte";
  import { getPosts } from "$lib/services/posts.js";

  /**
   * @type {import('@sveltejs/kit').PageLoad}
   */
  export async function load(event) {
    const { parent } = event;
    await parent();

    let data = {
      posts: [],
      postCount: 0,
      page: 1,
      totalPage: 1,
    };

    await loadSidebar(event);

    await getPosts({ page: event.params.page || 1, request: event }).then(
      (body) => {
        if (body.error) {
          return;
        }

        data = body;
      }
    );

    return { ...data, sidebar: HomeSidebar };
  }
</script>

<script>
  import { goto } from "$app/navigation";

  import Pagination from "$lib/component/Pagination.svelte";
  import Posts from "$lib/component/Posts.svelte";

  export let data;

  function reloadData(page = data.page) {
    getPosts({ page }).then((body) => {
      if (body.result === "ok") {
        if (page !== data.page) {
          goto(page === 1 ? "/" : "/blog/page/" + page);
        } else {
          data = body;
        }
      } else if (body.error === "PAGE_NOT_FOUND") {
        reloadData(page - 1);
      }
    });
  }
</script>
