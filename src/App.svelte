<script lang="ts">
  import { initClient, mutation, operationStore, query } from "@urql/svelte";

  interface Post {
    id: string;
    title: string;
    body: string;
    author: string;
    updatedAt: string;
    createdAt: string;
  }

  let name: string;

  name = "titouan";

  initClient({
    url: "http://localhost:3000/graphql",
  });

  let id: number;
  let title: string;
  let body: string;
  let author: string;

  const allPostsQuery = `
    query {
      allPosts {
        id
        title
        body
        author
        updatedAt
        createdAt
      }
    }
  `;

  const singlePostQuery = `
    query SinglePost($id: ID!) {
      post(id: $id) {
        id
        title
        body
        author
        updatedAt
        createdAt
      }
    }
  `;

  const createPostMutation = `
    mutation CreatePostMutation($title: String!, $body: String!, $author: String!) {
    createPost(input: { title: $title, body: $body, author: $author }) {
      id
      title
      body
      author
      updatedAt
      createdAt
    }
  }
  `;

  const updatePostMutation = `
    mutation UpdatePostMutation($id: ID!, $title: String!, $body: String!, $author: String!) {
    updatePost(input: { id: $id, title: $title, body: $body, author: $author }) {
      id
      title
      body
      author
      updatedAt
      createdAt
    }
  }
  `;

  const deletePostMutation = `
    mutation DeletePostMutation($id: ID!) {
    deletePost(input: { id: $id }) {
      id
      title
      body
      author
      updatedAt
      createdAt
    }
  }
  `;

  const allPosts = operationStore(allPostsQuery);

  const singlePost = operationStore(singlePostQuery, { id });
  $singlePost.variables = {
    id,
  };

  const createPostStore = operationStore(createPostMutation);

  const mutateCreatePost = mutation(createPostStore);

  function createPost(newTitle: string, newBody: string, newAuthor: string) {
    mutateCreatePost({
      title: newTitle,
      body: newBody,
      author: newAuthor,
    });
  }

  const updatePostStore = operationStore(updatePostMutation);

  const mutateUpdatePost = mutation(updatePostStore);

  function updatePost(
    id: number,
    newTitle: string,
    newBody: string,
    newAuthor: string
  ) {
    console.log(id);

    const post = query(singlePost);

    console.log(post);

    mutateUpdatePost({
      title: newTitle,
      body: newBody,
      author: newAuthor,
    });
  }

  const deletePost = operationStore(deletePostMutation, { id });

  query(allPosts);
  query(singlePost);

  const updateId = () => {
    $singlePost.variables.id = id;
    $singlePost.reexecute();
  };
</script>

<main>
  <h1>Hello {name}!</h1>

  {#if $allPosts.fetching}
    <p>loading...</p>
  {:else if $allPosts.error}
    <p>Oh no... {$allPosts.error.message}</p>
  {:else}
    <ul>
      {#each $allPosts.data.allPosts as post}
        <li>
          {post.id}
          {post.title}
          {post.body}
          {post.author}
        </li>
      {/each}
    </ul>
  {/if}

  {id}
  {#if $singlePost.fetching}
    <p>loading...</p>
  {:else if $singlePost.error}
    <p>{$singlePost.error.message}</p>
  {:else}
    {$singlePost.data.post.id}
    {$singlePost.data.post.title}
    {$singlePost.data.post.description}
    {$singlePost.data.post.author}
    {$singlePost.data.post.createdAt}
    {$singlePost.data.post.updatedAt}
  {/if}

  <div>
    <input bind:value={id} type="number" placeholder="Enter the post ID" />
    <button type="submit" on:click={() => updateId()}> Search Post </button>
  </div>

  <p>{title}</p>

  <div>
    <input bind:value={title} type="text" placeholder="Enter Post Title" />
    <input bind:value={body} type="text" placeholder="Enter Post Body" />
    <input bind:value={author} type="text" placeholder="Enter Post Author" />
    <button type="submit" on:click={() => createPost(title, body, author)}>
      Create Post
    </button>
    <button type="submit" on:click={() => updatePost(id, title, body, author)}>
      Update Post
    </button>
  </div>
</main>

<style>
  main {
    text-align: center;
    padding: 1em;
    max-width: 240px;
    margin: 0 auto;
  }

  h1 {
    color: #ff3e00;
    text-transform: uppercase;
    font-size: 4em;
    font-weight: 100;
  }

  @media (min-width: 640px) {
    main {
      max-width: none;
    }
  }
</style>
