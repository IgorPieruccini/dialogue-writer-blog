---
title: Prototyping
date: 2020-10-13 21:07:10
tags:
---

My very first step for this project was creating a prototype to define the frameworks I will use plus the ones I want to learn _(Next js, Mongodb, Graphql)_. Since i don't have much experience with those libraries, the prototype it's a way of playing, mixing and learning how and what I can do, and mostly important if my idea for the project is feasible.

## The idea

My focus at this stage was learning how to integrate DynamoDB with Next js and create the most simple text editor which the user can type and in run time parse the text into a form that later on will have validations so the game developers can export the structure data to use in their game.

I started by creating a function that takes a text and return a javascript object containing:

```ts
{
    title: string,
    subTitle: string,
    dialogues: {
        character: string,
        dialogue: string
    }
}
```

The title is defined by wrapping by **[title]** and the subtitle must start with **##**.
The dialogues are simple separate by one or more line breaks, and inside each dialogue the parse looks for a **:** to define what's a character and dialogue text.

Here is how the parser looks like:

```ts
export const parseDialogueDocument = (text: string): DocumentProps => {
  const textWithOutTitleAndSubTitle = Text(text).removeTitle().removeSubTitle()
    .value;

  const sections = getDialogueSections(textWithOutTitleAndSubTitle);
  const title = getTitle(text)?.[0];
  const subtitle = getSubtitle(text)?.[0];

  const dialogues = sections
    .filter((section) => !!getCharacterAndDialogue(section))
    .map((section) => getCharacterAndDialogue(section));

  return {
    title: Text(title ?? "").removeTitleTag().value,
    subTitle: Text(subtitle ?? "").removeSubTitleTag().value,
    dialogues,
  };
```

### The result

Here is a gif showing how the text translate visually to a javascript object. For this effect I create a component that reads the object and creates the UI.
For preventing the parser to execute on every change I added a debounce that will only execute after 3 seconds the user is inactive (not typing).

![](/gifs/dialogue-writer-test.gif)

### Fetching and Saving with MongoDB

Next js makes thing really ease for us, by creating any file inside the `pages` folder next js creates a router accessible on `http:localhost:3000/{filename}`, so composing a router is as easy as creating files and directories.

Since i needed to show different text dialogue depending on the **dialogue id**, for example: `http:localhost:3000/dialogues/{dialogue_id}`, I used something called **dynamic routing**, so this way I can get the id and fetch the text from mongo db.

To create a dynamic routing you just need to set the name of your file as `[id].ts`, and the id will be accessible by using a function called `getStaticProps`, Next.js will pre-render this page at build time using the props returned by getStaticProps.

Here you can see what I did, in this case i fetch the text calling getDialogue and return it inside a props:.

```ts
//[id].ts
export const getStaticProps = async ({ params: { id } }) => {
  const text = await getDialogue().byId(id);

  return {
    props: {
      text,
      id,
    },
  };
};

// mongodb.utils.ts
export const getDialogue = () => ({
  byId: async (_id: string) => {
    const dialogue = await collection.findOne({ _id });
    if (!dialogue) return null;
    return dialogue.text;
  },
  all: async () => {
    const dialogues = await collection.find({});
    return JSON.stringify(dialogues);
  },
});
```

By doing that, props is passed to you component, example:

```ts
const Dialogue = ({ text, id }) => {
    ....
    return <div>...<div>
}
```

## Conclusion

I tried to share a little bit of my first experience with next js and mongoDB, their is a lot to learn and test, although this prototype was a good start.
Here is the list of libraries I decided to use to help me building this project:

- [antd](https://ant.design/)
- [formik](https://formik.org/)
- [yup](https://github.com/jquense/yup)
- [hapi/joi](https://www.npmjs.com/package/@hapi/joi)
- [@apollo/client](https://www.apollographql.com/docs/react/)
- [monk](https://automattic.github.io/monk/docs/GETTING_STARTED.html)
