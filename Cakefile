require "sweetbread"

# ── Workshop offerings (source/data/workshops.json) ───────────────────────
# Single source of truth for the ten bookable workshops — name, photo,
# copy, price, and group size. Two ways to pull them into a page:
#
#   <!-- WORKSHOP OFFERINGS -->
#     All ten, grouped by duration. Used on the main "host a workshop" page.
#
#   <!-- WORKSHOPS: berry-boat, moon-and-star{price:95}, tiny-trug -->
#     Any subset, in that order — one, five, ten, whatever an event needs.
#     Append {key:value, ...} after an id to override just that field for
#     just this page (price is the common one) without touching the shared
#     copy. Ids are the "id" field in workshops.json (the name, lowercased
#     and hyphenated).

loadWorkshops = ()->
  data = JSON.parse read "source/data/workshops.json"
  byId = {}
  for group in data.durations
    for offering in group.offerings
      byId[offering.id] = Object.assign {}, offering, durationLabel: group.label
  {data, byId}

# Splits on `sep` but ignores separators inside {...}, so an override block
# like {price:95,maxPeople:15} doesn't get torn apart at its own comma.
splitTopLevel = (str, sep)->
  parts = []
  depth = 0
  current = ""
  for char in str
    if char is "{" then depth += 1
    if char is "}" then depth -= 1
    if char is sep and depth is 0
      parts.push current
      current = ""
    else
      current += char
  parts.push current
  parts

# "id{key:value,key:value}" -> [id, {key: value, ...}]. Values that look
# numeric become numbers; everything else stays a string.
parseWorkshopRef = (ref)->
  match = ref.trim().match /^([a-z0-9-]+)(?:\{(.*)\})?$/
  throw new Error "Malformed workshop reference: \"#{ref}\"" unless match
  id = match[1]
  overrides = {}
  if match[2]
    for pair in splitTopLevel match[2], ","
      [key, value] = pair.split(":").map (s)-> s.trim()
      overrides[key] = if value isnt "" and not isNaN(Number(value)) then Number(value) else value
  [id, overrides]

# Renders one .hw-offering article. showDuration adds a third "3 hours"
# style pill to the terms line — needed for a custom subset, since it has
# no <h3> grouping to say that instead.
renderOffering = (o, {showDuration = false} = {})->
  note = if o.note then "\n              <p class=\"hw-note\">#{o.note}</p>" else ""
  duration = if showDuration then " <span>#{o.durationLabel}</span>" else ""
  """
          <article class="hw-offering">
            <figure><img src="#{o.image}" loading="lazy" alt="#{o.alt}" /></figure>
            <div class="hw-offering-body">
              <h4>#{o.name}</h4>
              <p>#{o.description}</p>#{note}
              <p class="hw-terms"><span>$#{o.price}</span> <span>#{o.minPeople} to #{o.maxPeople} people</span>#{duration}</p>
            </div>
          </article>
  """

# The full catalogue, grouped by duration — the <!-- WORKSHOP OFFERINGS --> marker.
renderAllOfferings = (data)->
  groups = data.durations.map (group)->
    offerings = group.offerings.map((o)-> renderOffering(o)).join "\n\n"
    """
        <h3 class="hw-duration">#{group.label}</h3>
        <div class="hw-offerings">
    #{offerings}
        </div>
    """
  groups.join "\n\n"

# A specific subset, in order, with optional per-instance overrides —
# the <!-- WORKSHOPS: id, id{price:95} --> marker.
renderCustomWorkshops = (byId, list)->
  refs = splitTopLevel list, ","
  cards = refs.map (ref)->
    [id, overrides] = parseWorkshopRef ref
    base = byId[id]
    throw new Error "Unknown workshop id \"#{id}\" — check source/data/workshops.json" unless base
    renderOffering Object.assign({}, base, overrides), showDuration: true
  offerings = cards.join "\n\n"
  """
      <div class="hw-offerings">
  #{offerings}
      </div>
  """

task "start", "Build, watch, and serve.", ()->
  invoke "build"
  invoke "watch"
  invoke "serve"

task "build", "Compile everything", ()->
  rm "public"

  compile "static", "source/**/*.!(html|css|coffee|json)", (path)->
    copy path, replace path, "source/": "public/"

  template = read "source/pages/_template.html"
  {data, byId} = loadWorkshops()
  allOfferings = renderAllOfferings data

  compile "pages", "source/pages/**/[!_]*.html", (path)->
    content = read path
    dest = replace path,
      "source/pages": "public"
      ".html": "/index.html"
      "/index/": "/" # fixes /index/index.html
    content = template.replace "PAGE CONTENT GOES HERE", content
    content = content.replace "<!-- WORKSHOP OFFERINGS -->", allOfferings
    content = content.replace /<!-- WORKSHOPS:\s*(.+?)\s*-->/g, (match, list)->
      renderCustomWorkshops byId, list
    write dest, content

  compile "global styles", ()->
    write "public/styles.css", concat readAll "source/styles/**/*.css"

task "watch", "Recompile on changes.", ()->
  watch "source", "build", reload

task "serve", "Spin up a live reloading server.", ()->
  serve "public"
