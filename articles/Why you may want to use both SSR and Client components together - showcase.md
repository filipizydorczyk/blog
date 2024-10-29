Last week I tried [NEXT.JS](https://nextjs.org/). My intention was to see what this framework can really do, especially when it comes down to SSR. To do it, I took my old idea for a project and decided to create its POC using the framework. The idea for a project was simple. User can add URLs to any website serving a content (it could be social media profile, news service or really any website serving content), and then he can use the feed to get the feed in single place instead of using multiple websites/accounts. As I mentioned it's just POC, so this app obviously does not do that, but it showcases the way it could be done which is having a headless browser that will scrape the content for you and display it in the app. While it was just a POC and decided to keep the data in local storage, instead of setting up the database, which turned out to be a fun showcase of how server components and client components can act together and serve different purposes. Here is why:

<img src="https://cms.filipizydorczyk.pl/api/v1/media/ssr.png">

1. First we are rendering the page skeleton. I will be by default pre-rendered by server, but `SubscriptionFeed` will be rendered only on the client. It will never be rendered on the server because we specifically told it not to. The reason why we wanted it to be a server component is because it uses local storage, and you can't use it on the server. It's simply not accessible out of the browser.

```js
const SubscriptionFeed = dynamic(() => import("../components/SubscriptionFeed"), {
  ssr: false,
  loading: () => <Loader />,
});

export default function Feed() {
  return (
    <>
      <SubscriptionFeed />
    </>
  );
}
```

2. The `SubscriptionFeed` will load the data from local storage using `getSubs()` hook and pass it to `SubscriptionFetcher` which is the server component now. The reason why it's loaded dynamically because we want to render it only once we fetched `listOfSubscriptions`. Since the cost of rendering this component is high, we want to only render it once we have a list we want to display.

```js
"use client";

const SubscriptionFetcher = dynamic(() => import("./SubscriptionFetcher"), {
  loading: () => <Loader />,
});

export default function SubscriptionFeed() {
  const { listOfSubscriptions, invalidSubscriptions } = getSubs();
  return (
    <article>
      <aside>
        <p>This subscriptions are not suported yet:</p>
        <ul>
          {invalidSubscriptions.map((sub) => (
            <li key={sub}>{sub}</li>
          ))}
        </ul>
      </aside>
      {listOfSubscriptions && <SubscriptionFetcher subs={listOfSubscriptions} />}
    </article>
  );
}
```

3. Then we have `SubscriptionFetcher`. This will iterate through the list provided by the client, do the web scraping part and render the feed view. The reason why it has to be server side component is because we are running headless Chrome to do the scraping, and it can't work in the browser. It has to be done on the server, therefore it needs to be server component rendered only when we have necessary data ready.

```js
"use server";

import { ChromeBrowser } from "../lib/browser";

export default async function SubscriptionFetcher(props: { subs: string[] }) {
  const subs = props?.subs || [];
  let feed: {
    link: string;
    img: string;
  }[] = [];

  await Promise.all(
    subs.map(async (sub) => {
      const browser = new ChromeBrowser();
      await browser.init();
      const data = (await browser.fetchTheFeed(sub))?.filter((sub) => !!sub.img) || [];
      feed = [...feed, ...data];
      await browser.close();
    })
  );

  return (
    <div style={{ display: "flex", gap: "1rem", marginTop: "2rem", flexWrap: "wrap" }}>
      {feed.map((im, index) => (
        <a href={im.link} key={index} target="_blank">
          <img style={{ borderRadius: "1rem", cursor: "pointer" }} src={im.img} />
        </a>
      ))}
    </div>
  );
}
```

This is how it worked together:

<video controls>
    <source src="https://cms.filipizydorczyk.pl/api/v1/media/ssr.mp4" type="video/mp4">
    Your browser does not support the video tag.
</video>
