require 'pathname'
require 'pry'
require 'rainbow'
require 'open4'

def project_path
  Pathname.new(File.dirname(__FILE__))
end

def build_path
  project_path + "build"
end

def output_path(format = :html)
  build_path + format.to_s
end

def output_file(format = :html, ext = :html)
  output_path(format) + output_filename(ext)
end

def output_filename(ext = :html)
  "book.#{ext}"
end

def formats
  %w(html pdf epub mobi docbook)
end

namespace :book do
  task :prepare do
    formats.each do |f|
      FileUtils.mkdir_p output_path(f)
    end
  end

  task :build, [:format, :ext, :command, :extra_args, :copy_images] => :prepare do |t, args|
    puts "Creating #{args[:format]} book"
    attrs = {
      'stylesheet' => 'stylesheets/book.css',
      'pdf-fontsdir' => 'pdf-styles/fonts',
      'pdf-stylesdir' => 'pdf-styles',
      'pdf-style' => 'book'
    }
    command = ["bundle", "exec", args[:command], "book.ad",
           "-D", output_path(args[:format]).to_s, # destination directory
           "-o", output_filename(args[:ext]).to_s, # output file
           "-B", project_path.to_s].join(" ") # Project base path
    attrs.each do |key, value|
      command += " -a #{key}=#{value}" # Extra attributes for asciidoctor
    end

    run_command(command)

    # Copy images, done for HTML versions
    if args[:copy_images]
      Dir[project_path + "ch*/images/**/*"].each do |source|
        target = source.gsub(project_path.to_s, output_path(args[:format]).to_s)
        FileUtils.mkdir_p(File.dirname(target))
        FileUtils.cp(source, target)
      end
    end
    puts "#{args[:format]} book created in #{output_file(args[:format], args[:ext])}"
  end

  desc 'Build HTML format'
  task html: [:scss] do
    Rake::Task["book:build"].invoke("html", "html", "asciidoctor", "", true)
  end

  task pdf: [:scss] do
    Rake::Task["book:build"].invoke("pdf", "pdf", "asciidoctor-pdf", "")
  end

  task epub: [:scss] do
    Rake::Task["book:build"].invoke("epub", "epub", "asciidoctor-epub3", "")
  end

  task mobi: [:scss] do
    Rake::Task["book:build"].invoke("mobi", "mobi", "asciidoctor-epub3", "-a ebook-format=kf8")
  end

  task docbook: [:scss] do
    Rake::Task["book:build"].invoke("docbook", "xml", "asciidoctor", "-b dockbook")
  end

  task :scss do
    run_command("scss stylesheets/book.scss stylesheets/book.css")
  end

  task :all => formats do
    puts "Created all formats"
  end

  task :clean do
    `rm -rf #{build_path}`
  end

  def run_command(command)
    puts Rainbow("Running:").green
    puts Rainbow("#{command}").green
    pid, stdin, stdout, stderr = Open4.popen4(command)
    ignored, process_status = Process.waitpid2(pid)

    if process_status.exitstatus == 0
      puts Rainbow("SUCCESS!").green
      return
    end

    puts Rainbow("Command failed:").red
    puts Rainbow(stderr.read).red
    exit(1)
  end
end

task :default => %w(book:clean book:html)
