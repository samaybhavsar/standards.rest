require 'faraday'
require 'json'
require 'yaml'

task default: [:download, :rewrite]

task :download do
  response = Faraday.new('http://webconcepts.info').get('/specs.json')
  raise "Cannot download webconcepts/specs.json" unless response.success?
  File.write('tmp/specs.json', response.body)
end



# Yield every object
def enumerate(data, wg = nil, &block)
  case data
  when Array then 
    data.each { |v| enumerate(v, wg, &block) }
  when Hash then 
    yield wg, data
    data.each { |k,v| enumerate(v, wg || k, &block) }
  end
end

task :rewrite do
  interest = YAML.load(File.read('./_data/interest.yml'))
  data = JSON.parse(File.read('./tmp/specs.json'))

  specs_by_uri = {}
  enumerate(data) do |wg, spec|
    next unless spec['URI']
    specs_by_uri[spec['URI']] = spec.merge('wg' => data[wg].slice('id', 'name', 'short'))
  end

  relevant_specs = interest['webconcepts'].map do |uri|
    spec = specs_by_uri[uri]
    raise "cannot find uri #{uri}" if spec.nil?
    spec
  end

  # Add our own standards in
  relevant_specs += interest['custom'].map do |spec|
    next if spec['wg'].nil?
    wg = data[spec['wg']]
    raise "cannot find a wg for #{spec['wg']}" unless wg
    spec.merge('wg' => wg.slice('id', 'name', 'short'))
  end

  relevant_specs.sort_by! { |spec| spec['title'] }

  File.write('_data/standards.yml', "# Generated by Rakefile, please do not edit\n\n" + YAML.dump(relevant_specs))
end
