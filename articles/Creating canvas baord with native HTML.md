There is so much software that allows you to edit graphics in 2D canvas. I mean software like Figma or Adobe XD. Usually the goal of these pieces of software is to provide a tool for some kind of visualizations, so typically implementations are done with some kind of canvas.

There are two issues I have with that. Most of the accessible tools are web based and proprietary, and very few allows me to store files locally. It's not an issue for things like website design for which I am using the tool only during the development, but if I want to create a mood board for something I am planning on starting in a few months, I would feel much safer having that on my drive than on someone's server. Also, these tools are restricted to some basic graphic components. You can embed HTML elements inside (at least to my best knowledge). No one seems to cry about that, but I figured if I do mood board for a small project, I might as well put there some additional components for list of tasks and some other stuff and turn it to a small organizational tool. Sounds good, but I wasn't able to find anything close to that.

I am not willing to spend time to develop a full application, but I was willing to create proof of concept. So... Objective: create Figma like 2D canvas that allows you to put HTML elements inside, move their position inside and zoom the entire thing. Additionally, use as little JS as I can. If I can use a native HTML solution, I should try that first.

First I thought about architecture. What components I am gonna to need. There are surely many ways to approach that, with one being these 3 concepts:

- **Viewport** - container in which we are going to draw
- **Board Space** - container to keep items in. Items position are going to be relative to that space. I'll explain why in a bit
- **Item Wrapper** - we might want to do various types of items, but all of them will keep track of its position and size. That's what item wrapper should be responsible for

<img src="https://cms.filipizydorczyk.pl/api/v1/media/react-board.gif">

I wanted to do most of the things with native solutions, so first thing I did is I made view port and port space. Viewport should have overflow set to scroll. In that way if board space will be bigger than a viewport we are going to be able to scroll in both axis. All without single line of JS. The only js we are going to add for now is to make it possible to scroll with mouse drag

```tsx
const [focusPoint, setFocusPoint] = useState<Cords>();
const [focusScrolling, setFocusScrolling] = useState<Cords>();

const focus: React.MouseEventHandler<HTMLDivElement> = (e) => {
  e.stopPropagation();
  setFocusPoint([e.clientX, e.clientY]);
  setFocusScrolling([e.currentTarget.scrollLeft, e.currentTarget.scrollTop]);
};

const unfocus: React.MouseEventHandler<HTMLDivElement> = (e) => {
  setFocusPoint(undefined);
};

const updateScrolling: React.MouseEventHandler<HTMLDivElement> = (e) => {
  if (focusPoint && focusScrolling) {
    e.currentTarget.scrollTop = focusScrolling[Y] + (focusPoint[Y] - e.clientY);
    e.currentTarget.scrollLeft =
      focusScrolling[X] + (focusPoint[X] - e.clientX);
  }
};

// Usage

<div
  onMouseDown={focus}
  onMouseUp={unfocus}
  onMouseMove={updateScrolling}
  style={styles}
/>;
```

I decided to do implementation with react, so all code samples are in react, but you can do all of that event with plain JavaScript.

It's also nice to be able to zoom in and out. Let's just use CSS property `transform: scale(1.0);` It should be added to board space. All we have to do is to change this property on zoom, and it will make the whole board space and its children bigger/smaller. If it's bigger than view port, we are going to be able to scroll thanks to view port overflow.
Last thing is item. For its position, we are going to use CSS `position: absolute;`, `top` and `bottom`. Of course, we have to be able to change that position, so we are going to add some JS to update it on drag.

```tsx
const styles: React.CSSProperties = {
  // styles ...
  transform: `scale(${zoom})`,
};

return <div style={styles}>{children}</div>;
```

And the last thing is resize. Believe it or not, but you can just set CSS for that

```tsx
const styles: React.CSSProperties = {
  resize: "both",
  border: "1px solid transparent",
  overflow: "auto",
};

return (
  <div style={styles} onMouseUp={onResizeHandler}>
    {children}
  </div>
);
```

If you want to save new size you need to add some JS to add callback on resize which will provide you with new `width` and `height`

```ts
const onResizeHandler = (e: React.MouseEvent<HTMLDivElement, MouseEvent>) => {
  const target = e.target as HTMLDivElement;

  onResize(id, [
    Number(target.style.width.replace("px", "")),
    Number(target.style.height.replace("px", "")),
  ]);
};
```

This is just POC, but it shows you how to implement the most important features, and you can start with that and then add new functionalities or improve existing ones. Items can be some HTML components or SVGs if you want to be able to add some basic shapes. Maybe someday I will do full implementation, but for now I at least know how I would start. Also, you might have multiple issues with events like with `onMouseMove` overlapping with other divs. To solve that, you can use `onMouseDown={(e) => e.stopPropagation()}`. You can see my [POC repo](https://github.com/filipizydorczyk/react-html-white-board/tree/master) for reference.
