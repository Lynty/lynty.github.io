# local dev

1. clone repo
1. `cd lynty.github.io/`
1. `sudo gem install bundler:2.2.30`
1. `bundle config set --local path 'vendor/bundle'`
1. `bundle install`
1. `bundle exec jekyll serve` or `rm .jekyll-metadata && rm -rf _site && bundle exec jekyll serve --incremental -o -l`

# to do

1. categorize posts by tags and link
1. add some javascript

# post topics
1. blockchain
    - different consensus mechanisms
    - upgrading smart contracts
1. anthos
1. kubernetes
1. proof of concepts
1. work stuff
1. career journey
1. dev workflows
1. productivity
1. go
1. setting up website
1. speeding up terraform, ansible, and pipeline execution
1. office construction
1. cloud run for anthos
1. takeaways from leading a project
  1. open pair programming room
  1. delegation
  1. architecture diagrams
  1. TDD vs slide deck

# pre-publish checklist
- [ ] check links
- [ ] final readthrough
- [ ] add tags
