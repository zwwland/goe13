### needlman-wunsch

```ruby

require 'json'

class NW2
  attr_accessor :seq1, :seq2, :gap, :data
  attr_reader :s, :lts, :lseq1, :lseq2, :matrix_trace

  def initialize(seq1, seq2, gap = 1)
    @seq1 = seq1
    @seq2 = seq2
    @gap = gap
    @s = '★'
    @data = []

    @matrix_score = Array.new(@seq1.length + 1) { Array.new(@seq2.length + 1) }
    @matrix_trace = Array.new(@seq1.length + 1) { Array.new(@seq2.length + 1) { [] } }
  end

  def result
    make_matrix
    d = {
      'lts' => '',
      'lseq1' => '',
      'lseq2' => '',
      'trace' => []
    }
    seq1 = "-#{@seq1}"
    seq2 = "-#{@seq2}"
    y = seq1.length - 1
    x = seq2.length - 1
    @data << d
    get_trace y, x, seq1, seq2, d
    @data.each { |a| a.each_value(&:reverse!) }
  end

  def print_matrix
    seq1 = "-#{@seq1}"
    seq2 = "-#{@seq2}"

    str = ' '
    (0...seq1.length).each do |a|
      str += format('%3s', seq1[a])
    end
    str += "\n"
    (0...seq2.length).each do |x|
      str += seq2[x].to_s
      (0...seq1.length).each do |y|
        str += format('%3s', @matrix_score[y][x])
      end
      str += "\n"
    end
    puts str

    puts ''

    str = ' '
    (0...seq1.length).each do |a|
      str += format('%3s', seq1[a])
    end
    str += "\n"
    (0...seq2.length).each do |x|
      str += seq2[x].to_s
      (0...seq1.length).each do |y|
        str += format('%3s', @matrix_trace[y][x][0])
      end
      str += "\n"
    end
    puts str
  end

  private

  def compare(char1, char2)
    if char1 == @gap || char2 == @gap || char1 != char2
      -1
    else
      1
    end
  end

  def make_matrix
    seq1 = "-#{@seq1}"
    seq2 = "-#{@seq2}"

    (0...(seq1.length)).each do |yy|
      (0...(seq2.length)).each do |xx|
        if yy.zero? && xx.zero?
          @matrix_score[yy][xx] = 0
          @matrix_trace[yy][xx] << '↖'
        elsif yy.zero?
          @matrix_score[yy][xx] = -1 * xx
          @matrix_trace[yy][xx] << '↑'
        elsif xx.zero?
          @matrix_score[yy][xx] = -1 * yy
          @matrix_trace[yy][xx] << '←'
        else
          c_score = @matrix_score[yy - 1][xx - 1] + compare(seq1[yy], seq2[xx])
          l_score = @matrix_score[yy][xx - 1] - @gap + 1
          t_score = @matrix_score[yy - 1][xx] - @gap + 1
          score = [c_score, l_score, t_score].max
          @matrix_score[yy][xx] = score
          @matrix_trace[yy][xx] << '←' if l_score == score
          @matrix_trace[yy][xx] << '↑' if t_score == score
          @matrix_trace[yy][xx] << '↖' if c_score == score
        end
      end
    end
  end

  def get_trace_branch(y, x, data, seq1, seq2, mt)
    case mt
    when '↖'
      data['trace'] << [y, x]
      data['lts'] += seq1[y]
      data['lseq1'] += seq1[y]
      data['lseq2'] += seq2[x]
      y -= 1
      x -= 1
    when '←'
      data['trace'] << [y, x]
      data['lts'] += seq1[y]
      data['lseq1'] += seq1[y]
      data['lseq2'] += @s
      y -= 1
    when '↑'
      data['trace'] << [y, x]
      data['lts'] += seq2[x]
      data['lseq1'] += @s
      data['lseq2'] += seq2[x]
      x -= 1
    end
    [y, x]
  end
end

def get_trace(y, x, seq1, seq2, data)
  while y.positive? || x.positive?
    mt = @matrix_trace[y][x]
    mt.each_index do |ei|
      d = ei.positive? ? JSON.parse(JSON.dump(data)) : data
      @data << d if ei.positive?
      yy, xx = get_trace_branch(y, x, d, seq1, seq2, mt[ei])
      get_trace(yy, xx, seq1, seq2, d)
    end
  end
end

n = NW2.new('B', 'ATGDF')


```