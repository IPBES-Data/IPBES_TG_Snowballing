# TODOs and TOADDs

## Including reports into the snowball search concept
>
> The snowball search relies on the data on OpenAlex (<https://openalex.org/works?sort=relevance_score%3Adesc&column=display_name,publication_year,type,open_access.is_oa,cited_by_count&page=1&filter=default.search%3ARoutledge%20Handbook%20of%20Environmental%20Accounting>)
>
> The problem with books is, how they are cited in the references (DOI, ISBN, without any, only the chapter, etc - many possibilities). Consequently, the number of works citing a book are not that reliable. In addition, the extraction of the references is also not that easy to be done automatically, as there are references per chapter, non standard formats, full text access is needed and text analysis, unless the publishers provide that info. So the data of works cited in the book is also not complete. I am doing the snowball search using only DOIs as identifiers for the key-works, wherefore that book is in there (as it has a DOI) but others are not.
>
> I would suggest to keep it as it is, as manual supplementation of the snowball results with the works cited in the article would again put a bias in concerning the works cited I the book / chapter, and leaving completely out works which cite the book / chapter.
>
> I would suggest to
> I include the book / chapter as a key-paper and do the snowball search.
> I supplement the first snowball search results with the works cited in the chapter / book which are not yet in the snowball results (nodes (papers) and edges (links in graphs) will be added).
>
> It needs to be noted, that this would only be done for the first snowball (S1). Subsequent snowball searches would be done only by using OpenAlex in an automated fashion.
>
> NB: this inclusion of the book / chapter as a key-paper and the supplementation with the cited articles, will lead to a bias towards the cited literature compared to the citing literature, i.e. reflecting on what the book / chapter is based and underestimating it’s imp[act, i.e. where it has been cited.
>
> Actually: by using this semi-automated process by effectively building by hand the snowball from one key-paper (in this case book / chapter), would make it possible to include reports as well, but with the same caveats and biasses. The value of doing this for any document (report / book / chapter / …) would depend on many factors, including questions to be asked (impact can not be assessed, but provenance can be analysed), age of the document (for a new document, the times it has been cited is not that reliable), ...
