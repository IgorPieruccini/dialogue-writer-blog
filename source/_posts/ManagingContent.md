---
title: Building the Dashboard
date: 2020-10-26 21:52:14
tags:
---

## Result

![](/gifs/dashboard.gif)

## Into details

One of the core features of this project is to Manage the **Games > Chapter > Episodes** that will hold many dialogue scripts,
for this part I started by creating each router/page and all react components with a dummy data to quickly test the navigation and check if it's matching the [Mockups](http://localhost:4000/mockup.html).

Having all pages and routers the next thing was creating the **graphql schemas** so I could predefine the data shape that my ui will consume.

```ts
`
  type Game {
    _id: String!
    name: String
    nChapters: Int
    nEpisodes: Int
    nCharacter: Int
  }

  type Chapter {
    _id: String!
    gameId: String
    name: String
    nEpisodes: Int
  }

  type Episode {
    _id: String!
    chapterId: String
    name: String
  }

  input GameInput {
    _id: String!
    name: String
    nChapters: Int
    nEpisodes: Int
    nCharacter: Int
  }

  input ChapterInput {
    _id: String!
    gameId: String
    name: String
    nEpisodes: Int
  }

  input EpisodeInput {
    _id: String!
    chapterId: String
    name: String
  }

  type Query {
    gameList: [Game]
    chapterList(gameId: String!): [Chapter]
    episodeList(chapterId: String!): [Episode]
  }

  type Mutation {
    createGame(content: GameInput): Game!
    removeGame(content: GameInput): Game!
    createChapter(content: ChapterInput): Chapter!
    removeChapter(content: ChapterInput): Chapter!
    createEpisode(content: EpisodeInput): Episode!
    removeEpisode(content: EpisodeInput): Episode!
  }
`;
```

To manage the mongoDB collections I've create a utils to help using the basic CRUD

```ts
// chapters example
const url = "mongodb://localhost:27017";
const db = monk(url);
const chapter = db.get("chapter");

export const chapterDB = () => ({
  get: {
    Id: async (_id: string) => {
      const data: Chapter = await chapter.findOne({ _id });
      if (!data) return null;
      return data;
    },
    all: async (gameId: string) => {
      const data = await chapter.find({ gameId: { $eq: gameId } });
      return JSON.stringify(data);
    },
  },
  create: async (data: Chapter) => await chapter.insert(omit(data, ["_id"])),
  remove: async (data: Chapter) => {
    const response = await chapter.remove({ _id: data._id });
    return JSON.stringify(response);
  },
});
```

so I can easily **get create or remove** any data.
with this utils ready I've created the resolvers _(Query & Mutation)_ that will be responsible to talk with mongoDB and return the response that the client need to manage the cache thought apollo client.

```ts
export const resolvers = {
  Query: {
    gameList: async () => {
      const data = await gameDB().get.all();
      return JSON.parse(data);
    },
    chapterList: async (_, { gameId }) => {
      const data = await chapterDB().get.all(gameId);
      return JSON.parse(data);
    },
    episodeList: async (_, { chapterId }) => {
      const data = await episodeDB().get.all(chapterId);
      return JSON.parse(data);
    },
  },
  Mutation: {
    createGame: async (_, data: { content: Game }) =>
      await gameDB().create(data.content),
    removeGame: async (_, data: { content: Game }) => {
      await gameDB().remove(data.content);
      return data.content;
    },
    createChapter: async (_, data: { content: Chapter }) =>
      await chapterDB().create(data.content),
    removeChapter: async (_, data: { content: Chapter }) => {
      await chapterDB().remove(data.content);
      return data.content;
    },
    createEpisode: async (_, data: { content: Episode }) =>
      await episodeDB().create(data.content),
    removeEpisode: async (_, data: { content: Episode }) => {
      await episodeDB().remove(data.content);
      return data.content;
    },
  },
};
```

With all that ready now the focus is the client side, so for starting I've create the queries & mutations using apollo client.

### Queries

```ts
export const GET_GAME_LIST = gql`
  query getGameList {
    gameList {
      _id
      name
      nChapters
      nEpisodes
      nCharacter
    }
  }
`;

export const GET_CHAPTER_LIST = gql`
  query getChapterList($gameId: String!) {
    chapterList(gameId: $gameId) {
      _id
      gameId
      name
      nEpisodes
    }
  }
`;

export const GET_EPISODE_LIST = gql`
  query getEpisodeList($chapterId: String!) {
    episodeList(chapterId: $chapterId) {
      _id
      chapterId
      name
    }
  }
`;
```

### Mutation

Here are only the mutation for game, if you want to check the rest check [here](https://github.com/IgorPieruccini/dialogue-writer/tree/master/apollo)

```ts
export const CREATE_GAME = gql`
  mutation createGame($content: GameInput) {
    createGame(content: $content) {
      _id
      name
      nChapters
      nEpisodes
      nCharacter
    }
  }
`;

export const REMOVE_GAME = gql`
  mutation removeGame($content: GameInput) {
    removeGame(content: $content) {
      _id
      name
      nChapters
      nEpisodes
      nCharacter
    }
  }
`;

 ... CREATE_CHAPTER
 ... REMOVE_CHAPTER
 ... CREATE_EPISODE
 ... REMOVE_EPISODE
```

For creating all those queries and mutations I've used graphql playground of apollo client,
running the project using _yarn dev_ you can access it throw [http://localhost:3000/api/graphql](http://localhost:3000/api/graphql)

![](/images/graphql-playground.png)
