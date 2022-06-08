### needlman-wunsch

```ruby

class NW2
  attr_accessor :seq1, :seq2, :gap
  attr_reader :s, :lts, :lseq1, :lseq2, :matrix_trace

  def initialize(seq1, seq2, gap = 1)
    @seq1 = seq1
    @seq2 = seq2
    @gap = gap
    @s = '★'

    @matrix_score = Array.new(@seq1.length + 1) { Array.new(@seq2.length + 1) }
    @matrix_trace = Array.new(@seq1.length + 1) { Array.new(@seq2.length + 1) { Array.new } }
  end

  def result
    make_matrix
    get_trace
  end

  def print_matrix
    seq1 = '-' + @seq1
    seq2 = '-' + @seq2

    str = ' '
    (0...seq1.length).each do |a|
      str += seq1[a] + ' '
    end
    str += "\n"
    (0...seq2.length).each do |x|
      str += seq2[x].to_s + ' '
      (0...seq1.length).each do |y|
        str += @matrix_score[y][x].to_s + ' '
      end
      str += "\n"
    end
    puts str

    puts ''

    str = ' '
    (0...seq1.length).each do |a|
      str += seq1[a] + ' '
    end
    str += "\n"
    (0...seq2.length).each do |x|
      str += seq2[x].to_s + ' '
      (0...seq1.length).each do |y|
        str += @matrix_trace[y][x][0].to_s + ' '
      end
      str += "\n"
    end
    puts str
  end

  private

  def compare(a, b)
    if a == @gap || b == @gap || a != b
      -1
    else
      1
    end
  end

  def make_matrix
    seq1 = '-' + @seq1
    seq2 = '-' + @seq2

    (0...(seq1.length)).each do |yy|
      (0...(seq2.length)).each do |xx|
        if yy == 0 && xx == 0
          @matrix_score[yy][xx] = 0
          @matrix_trace[yy][xx] << '↖'
        elsif yy == 0
          @matrix_score[yy][xx] = -1 * xx
          @matrix_trace[yy][xx] << '↑'
        elsif xx == 0
          @matrix_score[yy][xx] = -1 * yy
          @matrix_trace[yy][xx] << '←'
        else
          c_score = @matrix_score[yy - 1][xx - 1] + compare(seq1[yy], seq2[xx])
          l_score = @matrix_score[yy][xx - 1] - @gap
          t_score = @matrix_score[yy - 1][xx] - @gap
          score = [c_score, l_score, t_score].max
          @matrix_score[yy][xx] = score
          if l_score == score
            @matrix_trace[yy][xx] << '←'
          end
          if t_score == score
            @matrix_trace[yy][xx] << '↑'
          end
          if c_score == score
            @matrix_trace[yy][xx] << '↖'
          end
        end
      end
    end
  end

  def get_trace
    seq1 = '-' + @seq1
    seq2 = '-' + @seq2

    @lts = ''
    @lseq1 = ''
    @lseq2 = ''
    trace = []
    y = seq1.length - 1
    x = seq2.length - 1


    while y > 0 || x > 0
        mt = @matrix_trace[y][x]
      if mt.include? '↖'
        trace << [y, x]
        @lts += seq1[y]
        @lseq1 += seq1[y]
        @lseq2 += seq2[x]
        y -= 1
        x -= 1
      elsif mt.include? '←'
        trace << [y, x]
        @lts += seq1[y]
        @lseq1 += seq1[y]
        @lseq2 += @s
        y -= 1
      elsif mt.include? '↑'
        trace << [y, x]
        @lts += seq2[x]
        @lseq1 += @s
        @lseq2 += seq2[x]
        x -= 1
      end
    end
    @lts.reverse!
    @lseq1.reverse!
    @lseq2.reverse!

    puts @lts
    puts @lseq1
    puts @lseq2

    trace.reverse
  end
end

```