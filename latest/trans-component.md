# Trans Component

## Important note

While the Trans components gives you a lot of power by letting you interpolate or translate complex react elements - the truth is - in most cases you won't need it.

**As long you have no react nodes you like to be integrated into a translated text** \(text formatting, like `strong`, `i`, ...\) **or adding some link component - you won't need it** - most can be done by using the good old `t` function.

{% hint style="info" %}
Using the **t** function have a look at i18next documentation:

* [essentials](https://www.i18next.com/essentials.html)
* [interpolation](https://www.i18next.com/interpolation.html)
* [formatting](https://www.i18next.com/formatting.html)
* [plurals](https://www.i18next.com/plurals.html)
* ...
{% endhint %}

## Samples

### Using with react components

So you learned there is no need to use the Trans component everywhere \(the plain t function will just do fine in most cases\).

This component enables you to nest any react content to be translated as one string. Supports both plural and interpolation.

_Let's say you want to create following html output:_

> Hello **Arthur**, you have 42 unread messages. [Go to messages](../legacy-v9/trans-component.md).

**Before:** Your react code would have looked something like:

```javascript
<div>
  Hello <strong title="this is your name">{name}</strong>, you have {count} unread message(s). <Link to="/msgs">Go to messages</Link>.
</div>
```

**After:** With the trans component just change it to:

```javascript
<Trans i18nKey="userMessagesUnread" count={count}>
  Hello <strong title={t('nameTitle')}>{{name}}</strong>, you have {{count}} unread message. <Link to="/msgs">Go to messages</Link>.
</Trans>
```

_Your en.json \(translation strings\) will look like:_

```javascript
"userMessagesUnread": "Hello <1>{{name}}</1>, you have {{count}} unread message. <5>Go to message</5>.",
"userMessagesUnread_plural": "Hello <1>{{name}}</1>, you have {{count}} unread messages.  <5>Go to messages</5>.",
```

{% hint style="info" %}
[**saveMissing**](https://www.i18next.com/overview/configuration-options#missing-keys) will send a valid defaultValue
{% endhint %}

### Using for &lt;br /&gt; and other simple html elements in translations \(v10.4.0\)

{% hint style="info" %}
This was newly added in react-i18next@**v10.4.0**

Allows elements not having additional attributes like className and only no children \(void\) or one text child:

* &lt;br/&gt;  
* &lt;strong&gt;bold&lt;/strong&gt;  
* &lt;p&gt;some paragraph&lt;/p&gt;  

but not:

* &lt;i className="icon-gear" /&gt;  
* &lt;strong title="something"&gt;bold something&lt;/strong&gt;
{% endhint %}

It allows you to have basic html tags inside your translations which will get converted to valid react elements:

```jsx
<Trans i18nKey="welcomeUser">
  Hello <strong>{{name}}</strong>.
</Trans>
// JSON -> "welcomeUser": "Hello <strong>{{name}}</strong>.",
<Trans i18nKey="multiline">
  Some newlines <br/> would be <br/> fine
</Trans>
// JSON -> "multiline": "Some newlines <br/> would be <br/> fine"
```

You can use i18next.options.react to adapt this behaviour:

| option | default | description |
| :--- | :--- | :--- |
| transSupportBasicHtmlNodes | true | convert eg. &lt;br/&gt; found in translations to a react component of type br |
| transKeepBasicHtmlNodesFor | \['br', 'strong', 'i', 'p'\] | Which nodes not to convert in defaultValue generation in the Trans component. |

### Using with lists \(v10.5.0\)

You can use list.map as children. Mapping dynamic content.

```jsx
<Trans i18nKey="list_map">
  My dogs are named:
  <ul i18nIsDynamicList>
     {['rupert', 'max'].map(dog => (<li>{dog}</li>))}
  </ul>
</Trans>
// JSON -> "list_map": "My dogs are named: <1></1>"
```

Setting `i18nIsDynamicList` on the wrapping element will assert the nodeToString function creating the string for saveMissing will not contain children.

### Alternative usage

Depending on using [ICU as translation format](https://github.com/i18next/i18next-icu) it is not possible to have the needed syntax as children \(invalid jsx\). You can alternatively use the component like:

```javascript
<Trans
  defaults="hello <0>{{what}}</0>"
  values={{ what: 'world'}}
  components={[<strong>univers</strong>]}
/>
```

## How to get the correct translation string?

Guessing replacement tags _\(&lt;0&gt;&lt;/0&gt;\)_ of your component right is rather difficult. There are two options to get those translations directly generated by i18next.

1. use `debug = true` in i18next init call and watch your console for the missing key output 
2. use the [saveMissing feature](https://www.i18next.com/configuration-options.html#missing-keys) of i18next to get those translations pushed to your backend or handled by a custom missing key handler. 
3. understand how those numbers get generated from child index:

**jsx:**

```javascript
<Trans i18nKey="userMessagesUnread" count={count}>
    Hello <strong title={t('nameTitle')}>{{name}}</strong>, you have {{count}} unread message. <Link to="/msgs">Go to messages</Link>.
</Trans>
```

**results in string:**

```text
"Hello <1>{{name}}</1>, you have {{count}} unread message. <5>Go to message</5>."
```

**based on** the node tree**:**

```javascript
Trans.children = [
  'Hello ',                           // index 0: only a string
  { children: [{ name: 'Jan'  }] },   // index 1: element strong -> child object for interpolation
  ', you have',                       // index 2: only a string
  { count: 10 },                      // index 3: just object for interpolation
  ' unread messages. ',               // index 4
  { children: [ 'Go to messages' ] }, // index 5: element link -> child just a string
  '.'
]
```

**Rules:**

* child is a string =&gt; nothing to wrap just take the string  
* child is an object =&gt; nothing to to it's used for interpolation  
* child is an element: wrap it's children in &lt;i&gt;&lt;/i&gt; where i is the index of that element position in children and handle it's children with same rules \(starting element.children index at 0 again\)

## Trans props

<table>
  <thead>
    <tr>
      <th style="text-align:left"><em><b>name</b></em>
      </th>
      <th style="text-align:left"><em><b>type (default)</b></em>
      </th>
      <th style="text-align:left"><em><b>description</b></em>
      </th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">i18nKey</td>
      <td style="text-align:left">string (undefined)</td>
      <td style="text-align:left">
        <p>is optional if you prefer to use text as keys you can omit that and the
          translation will be used as a key.</p>
        <p></p>
        <p>can contain the used namespace by prepending it key in form <code>&apos;ns:key&apos;</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">ns</td>
      <td style="text-align:left">string (undefined)</td>
      <td style="text-align:left">namespace to use</td>
    </tr>
    <tr>
      <td style="text-align:left">t</td>
      <td style="text-align:left">function (undefined)</td>
      <td style="text-align:left">t function to use instead of i18next.t</td>
    </tr>
    <tr>
      <td style="text-align:left">count</td>
      <td style="text-align:left">integer (undefined)</td>
      <td style="text-align:left">optional count if you use a plural</td>
    </tr>
    <tr>
      <td style="text-align:left">tOptions</td>
      <td style="text-align:left">object (undefined)</td>
      <td style="text-align:left">optional options you like to pass to t function call (eg. context, postProcessor,
        ...)</td>
    </tr>
    <tr>
      <td style="text-align:left">parent</td>
      <td style="text-align:left">node (undefined)</td>
      <td style="text-align:left">a component to wrap the content into (default none, can be globally set
        on i18next.init) -&gt; needed for <b>react &lt; v16</b>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">i18n</td>
      <td style="text-align:left">object (undefined)</td>
      <td style="text-align:left">i18next instance to use if not provided</td>
    </tr>
    <tr>
      <td style="text-align:left">defaults</td>
      <td style="text-align:left">string (undefined)</td>
      <td style="text-align:left">use this instead of default content in children (useful when using ICU)</td>
    </tr>
    <tr>
      <td style="text-align:left">values</td>
      <td style="text-align:left">object (undefined)</td>
      <td style="text-align:left">interpolation values if not provided in children</td>
    </tr>
    <tr>
      <td style="text-align:left">components</td>
      <td style="text-align:left">array[nodes] (undefined)</td>
      <td style="text-align:left">components to interpolate based on index of tag &lt;0&gt;&lt;/0&gt;, ...</td>
    </tr>
  </tbody>
</table>## Additional options on i18next.init

```javascript
i18next.init({
  // ...
  react: {
    // ...
    hashTransKey: function(defaultValue) {
      // return a key based on defaultValue or if you prefer to just remind you should set a key return false and throw an error
    },
    defaultTransParent: 'div', // a valid react element - required before react 16
    transEmptyNodeValue: '', // what to return for empty Trans
    transSupportBasicHtmlNodes: true, // allow <br/> and simple html elements in translations
    transKeepBasicHtmlNodesFor: ['br', 'strong', 'i'], // don't convert to <1></1> if simple react elements
  }
});
```

{% hint style="warning" %}
Please be aware if you are using **React 15 or below**, you need to set the `defaultTransParent` or `parent` in props.
{% endhint %}

