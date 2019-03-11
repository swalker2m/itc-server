## mosaic-server

This is a partial fix for the issue where 2MASS tiles overlap the FOV in a non-helpful way, as described in Gemini issue `REL-1093`.

### Building and Running Locally

This requies a local install of my Montage fork on the `mArchiveList-segfault` branch. There was a [PR](https://github.com/Caltech-IPAC/Montage/pull/32) back to Montage that may have been merged by now, so check it before proceeding.

It also requires a patched OT with the change on my Ocs fork on the `tpe-fixes` branch, and a hacked `ImageCatalog.scala` thus:

```patch
@@ -88,7 +88,8 @@ abstract class AstroCatalog(id: CatalogId, displayName: String, shortName: Strin
   def adjacentOverlap: Angle = Angle.zero

   override def queryUrl(c: Coordinates, site: Option[Site]): NonEmptyList[URL] =
-    NonEmptyList(new URL(s" http://irsa.ipac.caltech.edu/cgi-bin/Oasis/2MASSImg/nph-2massimg?objstr=${c.ra.toAngle.formatHMS}%20${c.dec.formatDMS}&size=${size.toArcsecs.toInt}&band=${band.name}"))
+  //  NonEmptyList(new URL(s" http://irsa.ipac.caltech.edu/cgi-bin/Oasis/2MASSImg/nph-2massimg?objstr=${c.ra.toAngle.formatHMS}%20${c.dec.formatDMS}&size=${size.toArcsecs.toInt}&band=${band.name}"))
+    NonEmptyList(new URL(s" http://localhost:8080/?object=${c.ra.toAngle.formatHMS}%20${c.dec.formatDMS}&radius=${0.25}&band=${band.name}"))
 }
```

### Running on Heroku

This only works for Rob. Anyone else needs to be [added as a collaborator](https://devcenter.heroku.com/articles/collaborating#add-collaborators).

To release a new version to Heroku do:

```
sbt core/docker:publish
heroku container:release web
```

An example invocation is:

```
curl -o /tmp/foo.fits 'http://gemini-2mass-mosaic.herokuapp.com/?object=05:51:10.305%2008:10:21.43&radius=0.25&band=H'
```


### Next Steps

Next steps:

- Move cache to S3 … the dyno will run out of disk quickly
- I'm using fs2/cats-effect milestones in order to get `Bracket` and `Resource`. http4s isn't quite ready yet so I'm using Jetty for now on the front end.