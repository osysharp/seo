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

[Route("/k/{slug}")]
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

## Examples

Taken from this package's own tests, which compile and run on every release.

```osy
using Osysharp.Seo;

[Test]
void AProductWritesSchemaOrgJson_WithItsOfferAndNothingLeftNull() {
  var json = new SeoProduct {
    Name = "Blue Kettle",
    Image = ["https://kettles.example.com/blue.png"],
    Offers = new SeoOffer { Price = 349.50m, Currency = "SEK", Availability = SeoAvailability.InStock },
  }.ToJson();
  Assert.Equal("{\"@context\":\"https://schema.org\",\"@type\":\"Product\",\"name\":\"Blue Kettle\","
    + "\"image\":[\"https://kettles.example.com/blue.png\"],"
    + "\"offers\":{\"@type\":\"Offer\",\"price\":349.50,\"priceCurrency\":\"SEK\",\"availability\":\"https://schema.org/InStock\"}}",
    json);
}
```

From [`tests/seo.test.osy`](tests/seo.test.osy).

```osy
using Osysharp.Seo;

[Test]
void ABreadcrumbTrail_IsNumberedInOrder() {
  var json = new SeoBreadcrumbList { Steps = [
    new SeoBreadcrumb { Name = "Kettles", Url = "https://kettles.example.com/k" },
    new SeoBreadcrumb { Name = "Blue Kettle" },
  ] }.ToJson();
  Assert.Equal("{\"@context\":\"https://schema.org\",\"@type\":\"BreadcrumbList\",\"itemListElement\":["
    + "{\"@type\":\"ListItem\",\"position\":1,\"name\":\"Kettles\",\"item\":\"https://kettles.example.com/k\"},"
    + "{\"@type\":\"ListItem\",\"position\":2,\"name\":\"Blue Kettle\"}]}",
    json);
}
```

From [`tests/seo.test.osy`](tests/seo.test.osy).

```osy
using Osysharp.Seo;

[Test]
void SeveralBlocks_AreOneJsonArray_AndARawBlockIsItsOwnText() {
  var json = JsonLd.Of([
    new SeoOrganization { Name = "Kettle AB" },
    JsonLd.Raw("{\"@type\":\"Thing\"}"),
  ]).ToJson();
  Assert.Equal("[{\"@context\":\"https://schema.org\",\"@type\":\"Organization\",\"name\":\"Kettle AB\"},{\"@type\":\"Thing\"}]", json);
}
```

From [`tests/seo.test.osy`](tests/seo.test.osy).

## Learn more

- [The Osysharp.Seo reference](https://osysharp.com/reference/ui/seo-kit/)
- [Osysharp.Seo in the package catalogue](https://osysharp.com/packages/#osysharp-seo)
- [The Osy# reference](https://osysharp.com/reference/) — the language, the runtime and every kit
- [`llms-full.txt`](https://osysharp.com/llms-full.txt) — the whole reference and specification as one Markdown file, for a coding agent

## Licence

MIT — see [`LICENSE`](LICENSE).
