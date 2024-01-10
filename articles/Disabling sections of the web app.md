Sometimes it might happen that we need to disable a chunk of the website based on some condition. The way I was doing it for most of the time is I would just pass `disabled` attribute to the field or whole field set. But I encountered situations in which this is not enough. For example:
 - you want to disable not only fields but also links and images
 - you have inputs from different UI libraries, and you want all of them to have the same visual effect
 - the project structure has many nested components and there are some that needs to be disabled visually (like graying out the text)

In these situations, I would go with this simple chunk of CSS

```css
.test {
	pointer-events: none;
	cursor: default;
	opacity: 0.2;
}
```

This will make it visually not clickable and add some transparency. See this the example executed on YT main page.

<img src="https://cms.filipizydorczyk.pl/api/v1/media/YT disabled.png">
*Top section is disabled with mentioned chunk of CSS and bottom section is how it normally looks like*

Sometimes `pointer-events: none;` might not be enough so to make sure wrap the entire section with `fieldset` and add `border: none;`

```html
<fieldset disabled class="test" style="border: none;">
	<!-- All divs and inputs goes here -->
</fieldset>
```

This might be the quickest way to disable a big section of the website visually and functionally, and it can save even more time if the section contains multiple nested components.