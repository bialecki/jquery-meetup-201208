# Bootstrapping with HTML 5 data-* attributes and jQuery.data

## General idea

Often I'll have either components or entire pages with significant JavaScript functionality. The page loads with some semantic markup that describes the structure of the page and the data to display. Then when the page is loaded, scripts execute to parse and enhance the page. But there's an issue: where do I put configuration information that wouldn't be displayed on an unenhanced page?

For example, if I have a chart component that needs to be able to refresh itself, where do I put the URL that represents the resource that component should query to get updated data?

### Some solutions

#### Hard code resource URLs into my JavaScript

This is the quick and dirty solution, but it's brittle. It means if I change the location of that resource, things will probably break.

#### Use JavaScript variables or hidden fields

You can also stick this information in JavaScript variables an hidden fields, but they tend to add unnecessary complexity and feel messy. There's something better.

#### Use HTML 5 data-* attributes

This is the solution I've come to embrace. I'll put any information I need to page between the initial page and the enhanced JavaScript page via these attributes and then use jQuery to get easy access to them. Here's what it might look like:

    <div class="chart-component" data-refresh-data-uri="/path/to/refresh/data">
      <div class="chart-settings"></div>
      <canvas class="chart-canvas"></canvas>
    </div>

## Examples

So let's look at some examples.

### Auto-updating timestamps

See a full example at [http://jsfiddle.net/tL8t3/](http://jsfiddle.net/tL8t3/).

HTML:

    <div class="timestamp" data-timestamp="1345662976850">Just now</div>

JavaScript:

    var autoUpdateTimestamp = function (elem, timestamp, interval) {
      elem.text(timesince(timestamp));
      
      setTimeout(function () {
        autoUpdateTimestamp(elem, timestamp interval);
      }, interval);
    };

    $.fn.autoUpdate = function (interval) {
      return this.each(function () {
        var $this = $(this),
            timestamp = parseInt($this.data('timestamp'), 10);
            
        if (!timestamp) return;

        autoUpdateTimestamp($this, new Date(timestamp * 1000), interval);
      })
    }

    $('.timestamp').autoUpdate();

### A Backbone View (could represent a part of the page or the whole thing)

You can also take this idea up to larger components (even pages) and tie it directly into a framework like Backbone. There's a complete example at [http://jsfiddle.net/7yfAu/](http://jsfiddle.net/7yfAu/).

HTML:

    <div class="page" data-bootstrap-uri="/path/to/bootstrap/data/for/page"></div>

JavaScript:

    var BaseView = Backbone.View.extend({
        
        initialize: function () {
          $.ajax({
            type: 'GET',
            url: $el.data('bootstrap-uri'),
            success: _.bind(this.loaded, this)
          });
        },

        loaded: function (data, textStatus, xhr) {}

    });

    var ChartView = BaseView.extend({
      
      loaded: function (data) {
        this.model.set({
          'importantDetail' : data.importantDetail
        });

        this.fetchData(data.dataUrl);
      }

    });