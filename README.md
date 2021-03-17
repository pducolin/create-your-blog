# Create your blog

## Create a new Next.js app

```bash
npx create-next-app
```

## Add siteconfig.json

```json
{
  "title": "myblog.com",
  "description": "This is cool"
}
``` 

## Folder structure

```bash
components/
pages/
  post/
posts/
public/
  static/
```

## Homepage

```js
const Index = ({title, description}) => {
  return <div>Hello, world!</div>
}

export default Index

export const getStaticProps = async () => {
  const configData = await import("../siteconfig.json");

  return {
    props: {
      title: configData.default.title,
      description: configData.default.description,
    },
  };
};
```

## Components

* `Header.js`
* `Layout.js`
* `PostList.js`

### Layout

```js
import Head from "next/head";
export const Layout = ({ pageTitle, children }) => {
  return (
    <>
      <Head>
        <meta name="viewport" content="width=device-width, initial-scale=1" />
        <title>{pageTitle}</title>
      </Head>
      <section className="layout">
        <div className="content">{children}</div>
      </section>
      <footer>Built by me!</footer>
    </>
  );
};
```

And using it in `index.js`

```js
import { Layout } from "../components/Layout";

const Index = ({ title, description }) => {
  return (
    <Layout pageTitle={title}>
      <h1>Welcome to my blog</h1>
      <p>{description}</p>
      <main>Here we'll put the post list</main>
    </Layout>
  );
};

export default Index;

export const getStaticProps = async () => {
  const configData = await import("../siteconfig.json");

  return {
    props: {
      title: configData.default.title,
      description: configData.default.description,
    },
  };
};
```

### Header

```js
import Link from "next/link";

export const Header = () => {
  return (
    <>
      <header className="header">
        <nav className="nav">
          <Link href="/">
            <a>My Blog</a>
          </Link>
          <Link href="/about">
            <a>About</a>
          </Link>
        </nav>
      </header>
    </>
  );
};
```

And using it in `Layout.js`

```js
import Head from "next/head";
import { Header } from "./Header";

export const Layout = ({ pageTitle, children }) => {
  return (
    <>
      <Head>
        <meta name="viewport" content="width=device-width, initial-scale=1" />
        <title>{pageTitle}</title>
      </Head>
      <section className="layout">
        <Header />
        <div className="content">{children}</div>
      </section>
      <footer>Built by me!</footer>
    </>
  );
};
```

## Dynamic Routes

`pages/post/[postname].js`

### Add Markdown

```bash
yarn add react-markdown gray-matter raw-loader
```

Create `next.config.js`

```js
module.exports = {
  target: 'serverless',
  webpack: function (config) {
    config.module.rules.push({
      test: /\.md$/,
      use: 'raw-loader',
    })
    return config
  },
}
```

### [postname].js

```js
import { Layout } from "../../components/Layout";
import Link from "next/link";
import ReactMarkdown from "react-markdown";
import matter from "gray-matter";

export const BlogPost = ({ siteTitle, frontmatter, markdownBody }) => {
  if (!frontmatter) return <></>;

  return (
    <Layout pageTitle={`${siteTitle} | ${frontmatter.title}`}>
      <Link href="/">
        <a>Back to post list</a>
      </Link>
      <article>
        <h1>{frontmatter.title}</h1>
        <p>By {frontmatter.author}</p>
        <div>
          <ReactMarkdown source={markdownBody} />
        </div>
      </article>
    </Layout>
  );
};

export default BlogPost;

export const getStaticProps = async ({ ...ctx }) => {
  const { postname } = ctx.params;

  const content = await import(`../../posts/${postname}.md`);
  const config = await import(`../../siteconfig.json`);
  const data = matter(content.default);

  return {
    props: {
      siteTitle: config.title,
      frontmatter: data.data,
      markdownBody: data.content,
    },
  };
};

export const getStaticPaths = async () => {
  const blogSlugs = ((context) => {
    const keys = context.keys();
    const data = keys.map((key, index) => {
      let slug = key.replace(/^.*[\\\/]/, "").slice(0, -3);

      return slug;
    });
    return data;
  })(require.context("../../posts", true, /\.md$/));

  const paths = blogSlugs.map((slug) => `/post/${slug}`);

  return {
    paths,
    fallback: false,
  };
};
```

### My first post

```md
---
title: 'My first post'
author: 'Me'
---

## Header 2

I'm baby four dollar toast chambray health goth cold-pressed pinterest godard. Hashtag next level helvetica, glossier cronut prism hella flannel franzen flexitarian poutine organic. Actually pour-over hoodie direct trade humblebrag poutine ennui unicorn pinterest viral umami selfies snackwave skateboard. Fanny pack street art chicharrones, tbh selvage la croix pinterest knausgaard scenester.

### Header 3

Health goth cliche synth next level craft beer, ennui jean shorts tacos chambray lo-fi. Bicycle rights literally ugh hashtag tumblr unicorn. Selfies typewriter bespoke migas irony ennui dreamcatcher. Austin heirloom whatever fanny pack put a bird on it keytar kickstarter locavore swag listicle gochujang hoodie. Authentic portland sustainable farm-to-table, sriracha hoodie intelligentsia slow-carb cloud bread prism helvetica cray normcore. Irony authentic XOXO gentrify single-origin coffee literally.
```

Let's try it!

## List all posts

```js
import Link from 'next/link'

export const PostList = ({ posts }) => {
  if (posts === 'undefined') return null

  return (
    <div>
      {!posts && <div>No posts!</div>}
      <ul>
        {posts &&
          posts.map((post) => {
            return (
              <li key={post.slug}>
                <Link href={{ pathname: `/post/${post.slug}` }}>
                  <a>{post.title}</a>
                </Link>
              </li>
            )
          })}
      </ul>
    </div>
  )
}

export default PostList
```

Fetch post list in `index.js`

```js
export const getStaticProps = async () => {
  const configData = await import(`../siteconfig.json`)

  const posts = ((context) => {
    const keys = context.keys()
    const values = keys.map(context)

    const data = keys.map((key, index) => {
      let slug = key.replace(/^.*[\\\/]/, '').slice(0, -3)
      const value = values[index]
      const document = matter(value.default)
      return {
        frontmatter: document.data,
        slug,
      }
    })
    return data
  })(require.context('../posts', true, /\.md$/))

  return {
    props: {
      posts,
      title: configData.default.title,
      description: configData.default.description,
    },
  }
}
```

And `index` component becomes

```js
import matter from 'gray-matter'

import Layout from '../components/Layout'
import PostList from '../components/PostList'

const Index = ({ posts, title, description, ...props }) => {
  return (
    <Layout pageTitle={title}>
      <h1 className="title">Welcome to my blog!</h1>
      <p className="description">{description}</p>
      <main>
        <PostList posts={posts} />
      </main>
    </Layout>
  )
}

export default Index
```

## Push to Github

```bash
git remote add origin <github_repo_url>
git push --set-upstream origin main
```

## Deploy to Netflify

https://app.netlify.com/ 

## Tailwind CSS

```bash
yarn add --dev tailwindcss@latest postcss@latest autoprefixer@latest @tailwindcss/typography
npx tailwindcss init -p
```

```js
// tailwind.config.js
module.exports = {
  purge: ['./pages/**/*.{js,ts,jsx,tsx}', './components/**/*.{js,ts,jsx,tsx}'],
  darkMode: false, // or 'media' or 'class'
  theme: {
    extend: {},
  },
  variants: {
    extend: {},
  },
  plugins: [
    require('@tailwindcss/typography'),
    // ...
  ],
}
```

```js
// index.jsx
import "tailwindcss/tailwind.css";

...
```

Use it to style markdown

```js
<div className="prose mx-6 md:mx-8 lg:mx-12 max-w-none">
  <ReactMarkdown renderers={renderers}>{markdownBody}</ReactMarkdown>
</div>
```
