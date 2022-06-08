### needlman-wunsch

```ruby

class NW2
  attr_accessor :seq1, :seq2, :gap
  attr_reader :s, :lts, :lseq1, :lseq2

  def initialize(seq1, seq2)

    @seq1 = seq1
    @seq2 = seq2
    @gap = '-'
    @s = '★'

    @matrix_score = Array.new(@seq1.length + 1) { Array.new(@seq2.length + 1) }
    @matrix_trace = Array.new(@seq1.length + 1) { Array.new(@seq2.length + 1) }
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
    (0...seq1.length).each do |x|
      str += seq1[x].to_s + ' '
      (0...seq2.length).each do |y|
        str += @matrix_score[x][y].to_s + ' '
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
    (0...seq1.length).each do |x|
      str += seq1[x].to_s + ' '
      (0...seq2.length).each do |y|
        str += @matrix_trace[x][y].to_s + ' '
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
          @matrix_trace[yy][xx] = '↖'
        elsif yy == 0
          @matrix_score[yy][xx] = xx * -1
          @matrix_trace[yy][xx] = '←'
        elsif xx == 0
          @matrix_score[yy][xx] = yy * -1
          @matrix_trace[yy][xx] = '↑'
        else
          score = @matrix_score[yy - 1][xx - 1] + compare(seq1[yy], seq2[xx])
          trace = '↖'
          if @matrix_score[yy - 1][xx] + 1 > score
            score = @matrix_score[yy - 1][xx]
            trace = '↑'
          end
          if @matrix_score[yy][xx - 1] + 1 > score
            score = @matrix_score[yy][xx - 1]
            trace = '←'
          end
          @matrix_score[yy][xx] = score
          @matrix_trace[yy][xx] = trace
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
      if @matrix_trace[y][x] == '↖'
        trace << [y, x]
        @lts += seq1[y]
        @lseq1 += seq1[y]
        @lseq2 += seq2[x]
        y -= 1
        x -= 1
      elsif @matrix_trace[y][x] == '↑'
        trace << [y, x]
        @lts += seq1[y]
        @lseq1 += @s
        y -= 1
      elsif @matrix_trace[y][x] == '←'
        trace << [y, x]
        @lts += seq2[x]
        @lseq2 += @s
        x -= 1
      end
    end
    @lts.reverse!
    @lseq1.reverse!
    @lseq2.reverse!

    # puts @lts
    # puts @lseq1
    # puts @lseq2

    trace.reverse
  end

end

```