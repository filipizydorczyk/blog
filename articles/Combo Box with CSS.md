Recently I was requested to do a combo box component and I got the design. It was basically a select box with checkboxes inside that gives you multiple selection option. I didn't like the concept because there is no such component in HTML that I could easily style, and also not using plain HTML inputs makes it annoying to send this data to backend later.

<img style="max-width: 500px;margin: 0 auto;display: block;" src="https://cms.filipizydorczyk.pl/api/v1/media/combo-box-example.png">

_example of combo box from [vuetifyjs](https://vuetifyjs.com/en/components/combobox/#usage)_

After giving it some thought, I realized that I can just use checkboxes and send them to the backend with a right name (ex.`name="combobox.value1"`). So let's say I want a combo box with values 1,2,3. What I write is basically

```html
<label for="first"><input type="checkbox" name="combobox.first" /> 1</label>
<label for="second"><input type="checkbox" name="combobox.second" /> 2</label>
<label for="third"><input type="checkbox" name="combobox.third" /> 3</label>
```

It solves the issue with how I want to send the selected data. You would just get all selected names in payload. And to solve that visually, I have decided to create a div that would just display selected checkboxes names and show the options when we focus on that div. This is very minimal example for that

```html
<!DOCTYPE html>
<html>
  <head>
    <style>
      .cmp-wierd-select .cmp-wierd-select__options {
        display: none;
      }

      .cmp-wierd-select:focus-within .cmp-wierd-select__options {
        display: block;
      }
    </style>
  </head>
  <body>
    <div class="cmp-wierd-select">
      <input type="text" name="lol" />
      <div class="cmp-wierd-select__options" tabindex="-1">
        <label for="first"><input type="checkbox" name="first" /> 1</label>
        <label for="second"><input type="checkbox" name="second" /> 2</label>
        <label for="third"><input type="checkbox" name="third" /> 3</label>
      </div>
    </div>
  </body>
</html>
```

<img src="https://cms.filipizydorczyk.pl/api/v1/media/combo-box-css-example.gif">

Here we have text input instead of div, but you can do the same with div. You just need to remember to add `tabindex` attribute so that div is actually focusable `<div tabindex="-1">Tabbable due to tabindex.</div>`. The reason why we are using `focus-within` is because using just `focus` selector will close the box once we select any value (since he focuses is not on the preview element but on the checkbox). `focus-within` will remain active as long as the HTML element or any of its children is focused so clicking checkboxes will not close it.

And that is the perfect MVP of such a feature. It's very good for performance since we are not using JS. If you want to create a preview of selected values in the top input you will have to add some JS (`querySelectorAll(".classname input[type='checkbox']:checked")` might be useful) but it will not really impact the performance of the website since we already saved a lot of JS code. You might be even able to pretender all the badges and show/hide them with CSS rules instead of using JS. If anyone do that, let me know because I love skipping JS for the sake of the creative CSS rules.

Sources: [developer.mozilla.org/focus-within](https://developer.mozilla.org/en-US/docs/Web/CSS/:focus-within), [vuetifyjs.com](https://vuetifyjs.com/en/components/combobox/#usage), [developer.mozilla.org/tabindex](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/tabindex)
