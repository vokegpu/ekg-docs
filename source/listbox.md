# Listbox

The /listbox widget displays text fields as a 2D list. [Item](/ekg-docs/item) stores each text field, representing a geometry component.

# Container

Each row is a component container, a [listbox](listbox) container allocates all [Items](ekg-docs/item) cursively opened, and dynamically calculates the visible index, based on the opened components count.

When opening an [item](ekg-docs/item) component, the container increases the open counter, and when an [item](ekg-docs/item) component is closed, it decreases the open counter. This simple arithmetic system is useful for estimating the container height.

The rendering optimization is possible with a unique height, and by default, it is [font](ekg-docs/font) normal size. Calculating a dynamically visible index requires normalizing the scroll value with the estimated height. `scroll` means the scrolling value, `len` is the container components' length, and `height` is the container's estimated height.

![](https://cdn.discordapp.com/attachments/1064693858245546045/1170436090063228939/848372603294974024.png?ex=6559088d&is=6546938d&hm=ecdad3b26c03adcd8c8242fb44d9be96cc461356831cfa59a7038cb73696def1&)

The dynamic index calculation inverts the signal of the `scroll` because the scroll vector must subtract the position of the rectangle, and for the formula, it is not required, since we want to get normalized.

All closed components are skipped in the rendering section, the [item](item) component contains the count of total children, and it is used to increase the current index iteration.


