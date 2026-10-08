# Osysharp.Seo

schema.org structured data for a page's head: what a search engine reads to show a product's price and stock, a
breadcrumb trail, or a site's name and logo beside a link. Each class writes its own JSON-LD, including `@context` and
`@type`, and each is a `JsonLd` block, so `head { StructuredData = … }` takes it directly.

- **Products.** `SeoProduct` covers the name, pictures, SKU/GTIN and brand. Its `SeoOffer` holds the price, currency,
  availability, condition and sale end date, and its `SeoRating` the average of its reviews.
- **Articles.** `SeoArticle` is a help page, a guide or a post: its headline, its line, its language, when it was
  published and last changed. Say `Type = "TechArticle"` for a how-to.
- **Where a page sits.** `SeoBreadcrumbList` takes a list of `SeoBreadcrumb { Name, Url }` and numbers the steps for
  you.
- **Who and what.** `SeoOrganization` has the name, logo and profiles elsewhere. `SeoWebSite` has the site's name and
  address.
- **Anything else.** `JsonLd.Raw(json)` is JSON you wrote yourself, for any schema.org type the kit does not ship.
- **Safe whatever the data says.** A member left null is simply absent. The head escapes the JSON for the page, so a
  product named `</script>` stays a name.

## Use it

```osy
// app.osy
use Osysharp.Seo@0;
```

```osy
using Osysharp.Seo;

[Page("/k/{slug}")]
[AllowAnonymous]
component KettlePage(string slug) {
  var kettle = Kettle.Where(k => k.Slug == slug).FirstOrDefault();
  head {
    Title = kettle == null ? "Not found" : kettle.Name;
    StructuredData = kettle == null ? null : [
      new SeoProduct {
        Name = kettle.Name,
        Offers = new SeoOffer { Price = kettle.Price, Currency = "SEK", Availability = SeoAvailability.InStock },
      },
      new SeoBreadcrumbList { Steps = [ new SeoBreadcrumb { Name = "Kettles", Url = "https://kettles.example.com/k" },
                                        new SeoBreadcrumb { Name = kettle.Name } ] },
    ];
  }
  render { … }
}
```

The reference page is `osy docs seo kit`; the head itself is `osy docs head`.

## Source

`model/Seo.osy`: every class in one file, each one a few lines of fields with their schema.org names.
