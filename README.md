# vue_ebook

## Project setup

```shell
npm install
```

### change navigation.js source code

```javascript
unpack(toc, parent) {
    var item;
    for (let i = 0; i < toc.length; i++) {
        item = toc[i];
        if (item.href) {
            this.tocByHref[item.href] = {
                tocIndex: i,
                subtocIndex: null
            };
        }
        if (item.id) {
            this.tocById[item.id] = {
                tocIndex: i,
                subtocIndex: null
            };
        }
        this.length++;
        if (item.subitems.length) {
            this.unpackSub(item.subitems, i);
        }
    }
}
unpackSub(toc, parent) {
    for (let i = 0; i < toc.length; i++) {
        const subitem = toc[i];
        if (subitem.href) {
            this.tocByHref[subitem.href] = {
                tocIndex: parent,
                subtocIndex: i
            };
        }
        if (subitem.id) {
            this.tocById[subitem.id] = {
                tocIndex: parent,
                subtocIndex: i
            };
        }
        this.length++;
    }
}
```

### Compiles and hot-reloads for development

```shell
npm run serve
```

### Compiles and minifies for production

```shell
npm run build
```

### Lints and fixes files

```shell
npm run lint
```