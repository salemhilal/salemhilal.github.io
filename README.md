# salem.io: a humble site all about me

The site is just hand-coded HTML until it gets more complex, with
[Tachyons][tachyons] for styling.

## how do i do it

```sh
# First, always:
yarn install

# To develop with live-reloading, just run
yarn run dev
# and then visit localhost:8080.

# For a prod build, run
yarn run build
# If you want to see that build in your browser,
yarn run serve
```

## notes to self

To optimize my images, I run em through imagemagick. The following script:
* converts any pngs to jpgs
* turns transparent backgrounds white
* strips comments and profiles 
* uses [plane interlacing][interlace] to make our jpgs progressive

```sh
for i in ./articles/**/*.{jpg,png} ; do \
    echo "converting $i" && \
    convert -strip -interlace Plane -quality 85% -background white -flatten "$i" "${i%.*}.jpg" \
; done
```

I currently have to run this manually whenever I add images, but I'd love to make it part of eleventy.

[tachyons]: http://tachyons.io/
[interlace]: https://imagemagick.org/MagickStudio/Interlace.html
