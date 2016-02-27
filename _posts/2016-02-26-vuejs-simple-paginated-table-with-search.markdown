---
layout: post
title:  "Vuejs - Making a Simple Paginated Table with Search"
date:   2015-02-26 16:56:56
categories: software
---
### Some Background
At work, we use [jQuery DataTables plugin][data-tables] a good bit. The plugin is great for displaying static data in a table with some quick searching and sorting. But recently my team has started feeling the pain of an increasingly complex frontend, and jQuery alone wasn't solving our problems. In came [Vue.js][vue-js].

The problem is that Vue and DataTables do not play nice together. If you use Vue to control the insertion and deletion of rows in a table that is also hooked up to DataTables, you'll get some great errors about Vue not being able to find the parent container. This is because DataTables changes the markup underneath Vue.

You will also have problems in the other direction. As Vue removes and adds rows in the table, the DataTable controlled markup will not update. So you will have the footer on the table saying there are 100 rows, when there are really only 9.

All of this was obvious the moment I tried to mix the two, considering how different the two libraries paradigms are. So rather than mix the two and do awful forced updates, I looked into how easy it would be to create a simple version of DataTables with Vue. It turns out to be very easy!

### The Problem
I needed a very basic table with searching and pagination. That's it. No sorted columns, as the order needed to be controlled by some business rules, not the user. The table also has frequent inserts and deletes from user interactions.


### Solution
Full example [here][fiddle-first-example].

We will have two components: (1) the main ViewModel and (2) a child grid component that holds our table data. The ViewModel will control all our dynamic data and the grid component will be a dumb component with props only.

The ViewModel will control 5 attributes:
{% highlight html %}
data: {
  searchQuery: '', 		  // what the user types in the search box
  gridColumns: ['name', 'power'], // keys into our data objects for each column
  gridData: gridData, 		  // data for the table [{..}, {..}, ...]
  startRow: 0, 			  // row that our table will start with for pagination
  rowsPerPage: 10 		  // how many table rows we want in each page
}
{% endhighlight %}

The grid component will just take the gridData and gridColumns to display what our ViewModel passes down. Here it is in it's entirety (although missing a script tag with id="grid-template" because the syntax highlighter wasn't having it):

{% highlight html %}
<table>
  <thead>
    <tr>
      <th v-for="key in columns">
        {% raw %}{{key | capitalize}}{% endraw %}
      </th>
    </tr>
  </thead>
  <tbody>
    <tr v-for="entry in data">
      <td v-for="key in columns">
        {% raw %}{{entry[key]}}{% endraw %}
      </td>
    </tr>
  </tbody>
</table>
{% endhighlight %}

{% highlight javascript %}
Vue.component('grid', {
  template: '#grid-template',
  props: {
    data: Array,
    columns: Array,
  }
});
{% endhighlight %}

#### Search
Let's start with search. The [grid component example][vue-js-grid-component] that this work was based on already had it built in, but I'll quickly walk through it as some slight changes have to be made later on.

Vue makes searching a table incredibly easy with the filter concept. This all happens in the html of our ViewModel to the data we pass to our grid component:

{% highlight html %}
<grid :data="gridData | filterBy searchQuery" :columns="gridColumns" />
{% endhighlight %}

You can see we use the [filterBy][vue-js-filter-by] filter. It is a function in the Vue library that takes some text and looks in all the values of each of the objects in the array we are filtering. So it will search in each column of each row with no extra work on our part.

To get the search text, we make an input box and tie it to our searchQuery field in our ViewModel's data via the v-model directive. As the user types, the filter happens instantly and the table is updated.

{% highlight html%}
<form id="search">
  Search
  <input name="query" v-model="searchQuery">
</form>
{% endhighlight %}

#### Pagination
To handle pagination, we need to revisit our creation of the grid component above. We have another Vue filter that is perfect for our use case, [limitBy][vue-js-limit-by].

{% highlight html%}
<grid :data="gridData | filterBy searchQuery | limitBy rowsPerPage startRow" :columns="gridColumns" />
{% endhighlight %}

The limitBy filter takes two parameters. The first is the amount we want to limit and the second is the offset. For my use case, I hard coded 10 rows per page, but you could easily extend my example to let the user modify the amount of rows.

As far as changing the page, I didn't get very fancy. All I included was a back and next button.

{% highlight html%}
<div id="page-navigation">
  <button @click=movePages(-1)>Back</button>
  {% raw %}<p>{{startRow / rowsPerPage + 1}} out of {{gridData.length / rowsPerPage}}</p>{% endraw %}
  <button @click=movePages(1)>Next</button>
</div>
{% endhighlight %}

I include some click handlers to move pages, as well as a visual cue for the user to know what page they are on. But we need the actual click handler. For that, we include a new method in our methods section of the ViewModel's instantiation.

{% highlight javascript%}
methods: {
  movePages: function(amount) {
    var newStartRow = this.startRow + (amount * this.rowsPerPage);
    if (newStartRow >= 0 && newStartRow < gridData.length) {
      this.startRow = newStartRow;
    }
  },
  ...
}
{% endhighlight %}

#### Ordering
So I mentioned before that we would need to order the data based on some business rules. There is a built in [orderBy][vue-js-order-by] filter already created in Vue. Unfortunately it isn't quite good enough for our needs. Luckily, we can make our own [filters][vue-js-custom-filters]!

Rather than creating a global filter like the Vue docs show, I'm going to create a filter in the instantiation of my ViewModel. That way it is only scoped to my ViewModel and it's descendants.

{% highlight javascript%}
filters: {
  orderByBusinessRules: function(data) {
    return data.slice().sort(function(a, b) {
      // Your really complicated custom sorting logic here
      return a.power - b.power;
    });
  }
}
{% endhighlight %}

Notice how we slice() the array first to create a copy. In a filter, you do NOT want to mutate data. Especially here, a mutation of the data could cause an infinite loop where sorting the array causes Vue to run our sort again and again as it's observer notices changes. In fact Vue gives us a nice warning if we remove the slice: **[Vue warn]: You may have an infinite update loop for watcher with expression: gridData**.

We also need to update the data we pass to our grid component. We make sure to do the limitBy last.

{% highlight html %}
<grid :data="gridData | orderByBusinessRules | filterBy searchQuery | limitBy rowsPerPage startRow" :columns="gridColumns">
{% endhighlight %}

#### Some bugs
Here is the complete [example][fiddle-first-example]. Nice and simple, but you might notice a couple of bugs.

For instance, go to the third or fourth page and then type in Jackie Chan. That's right, you won't find him. To fix that issue, we will need to reset the pages to the first page when the user types in the search bar. This can be done by simply resetting the startRow on the keyup event of the search box.

{% highlight javascript%}
methods: {
  ...
  resetStartRow: function() {
    this.startRow = 0;
  }
},
{% endhighlight %}

{% highlight html%}
<input name="query" v-model="searchQuery" @keyup="resetStartRow">
{% endhighlight %}

That fixes that issue, but try typing in a search. It always shows 1 out of 10 pages, even though the results have been pared down. Fixing this issue required a bit more restructuring. For it, I moved the responsibility of the pagination into the grid component. I kept the search filtering and the ordering in the ViewModel, but let the grid component handle the limitBy.

First, we move the page-navigation into the grid component's template. Then the grid component performs the limitBy itself:

{% highlight html%}
<tr v-for="entry in data | limitBy rowsPerPage startRow">
{% endhighlight %}

We also need to pass the props the page navigation section needs to the grid component and remove the limitBy filter.
{% highlight html%}
<grid :data="gridData | orderByBusinessRules | filterBy searchQuery" :columns="gridColumns" :move-pages="movePages" :start-row="startRow" :rows-per-page="rowsPerPage">
{% endhighlight %}

Here is the [final solution][fiddle-final-example]. The restructuring solves the problem because the grid component is only given the filtered down and ordered data, which allows it to calculate the number of pages correctly.

### Conclusion
I hope you learned how easy Vue can make your life. So far I'm having a really great time sharing it with my team. I truly believe it will solve a lot of our javascript woes. But as the JS winds blow, only time will tell.

If you don't feel like rolling your own table, there is a nice [vue-tables][vue-tables] library I found that seems pretty robust although I haven't tried it out yet. It covers a lot more use cases and has column sorting built in.

[data-tables]: https://www.datatables.net/ 
[vue-js]: http://vuejs.org/
[vue-js-grid-component]: http://vuejs.org/examples/grid-component.html
[vue-js-filter-by]: http://vuejs.org/api/#filterBy
[vue-js-limit-by]: http://vuejs.org/api/#limitBy
[vue-js-order-by]: http://vuejs.org/api/#orderBy
[vue-js-custom-filters]: http://vuejs.org/guide/custom-filter.html
[fiddle-first-example]: http://jsfiddle.net/ctlevi/65ug9tu0/1/
[fiddle-final-example]: http://jsfiddle.net/ctlevi/xbarpj1L/3/
[vue-tables]: https://github.com/matfish2/vue-tables
