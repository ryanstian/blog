---
title: Group Pictures
description: NexT User Docs – NexT Supported Tags – Group Pictures
---

### Usage

```jinja
{% grouppicture [number]-[layout] %}{% endgrouppicture %}
<!-- Tag Alias -->
{% gp [number]-[layout] %}{% endgp %}
```

- `[number]` : *Optional parameter.* Total number of pictures to add in the group pictures.
- `[layout]` : *Optional parameter.* The index of the layout, which can be obtained according to the figure below. For example, if you want to apply the second layout to 4 pictures, then use

    ```jinja
    {% grouppicture 4-2 %}{% endgrouppicture %}
    ```

![Group Picture Layout](/images/group-picture-1.png)
![Group Picture Layout](/images/group-picture-2.png)

### Examples

```jinja
{% grouppicture 6-3 %}
![](/images/next.svg)
![](/images/next.svg)
![](/images/next.svg)
![](/images/next.svg)
![](/images/next.svg)
![](/images/next.svg)
{% endgrouppicture %}
```

{% grouppicture 6-3 %}
![](/images/next.svg)
![](/images/next.svg)
![](/images/next.svg)
![](/images/next.svg)
![](/images/next.svg)
![](/images/next.svg)
{% endgrouppicture %}

```jinja
{% gp 5-2 %}
![](/images/next.svg)
![](/images/next.svg)
![](/images/next.svg)
![](/images/next.svg)
![](/images/next.svg)
{% endgp %}
```

{% gp 5-2 %}
![](/images/next.svg)
![](/images/next.svg)
![](/images/next.svg)
![](/images/next.svg)
![](/images/next.svg)
{% endgp %}
