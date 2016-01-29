# Graviton.jl - Julia bindings for the GeneTrail2 web service.

This package provides Julia bindings for Graviton web services. An example of a
Graviton web service is the [GeneTrail2](https://genetrail2.bioinf.uni-sb.de)
server for the computation of (gene set) enrichments.

The package binds to the RESTful API provided by the server and allows to
seamlessly run Graviton jobs in a Julia environment.

## Documentation
Refer to the Julia documentation for detailed method descriptions.

 * Use `Session` to start a session with GeneTrail2.
 * Use `upload`, `upload_gmt`, and `download` to transfer data to and from the server.
 * Use `job_setup`, `job_start`, `job_stop`, `job_query`, and `job_wait` for starting computations.

Additional API documentation can be found at https://apidoc.bioinf.uni-sb.de/
and http://genetrail2.bioinf.uni-sb.de/help?topic=api.

## Example
The following function computes an enrichment using the unweighted
Kolmogorov-Smirnov statistics on scores computed using the shrinkage t-test.

```julia
import Graviton;

function ksenrichment(exprfile, sample_group, reference_group)
    session = Graviton.Session();

    data = Graviton.upload(session, exprfile);

    # Compute scores for the uploaded data
    scores = Graviton.compute_scores(data,
        sample_group,
        reference_group,
        "independent-shrinkage-t-test",
        printstatus=false
    );

    categories = Graviton.categories(
		organism=9606,
		 pipeline=Graviton.Pipeline.Gene
	);

    enrichment = Graviton.compute_enrichment(scores, "gsea", categories,
        adjustment="benjamini_hochberg",
        permutations=1000,
        pValueStrategy="row-wise"
    );

    Graviton.download(enrichment, "enrichment.zip");
end
```

## Known issues
 * Some functionality is missing.
 * The User type is not very useful right now.
 * The job interface is clunky.
 * The API is very path based and does not allow to pass streams in order to
   avoid unnecessary copies.
 * Changing the server URL is currently not possible.
