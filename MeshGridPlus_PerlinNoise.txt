@mod = Sketchup.active_model # Open model
@ent = @mod.entities # All entities in model
@sel = @mod.selection # Current selection
@view = @mod.active_view
@dizi = Array.new(1000) {Array.new(1000)}
@width=20
@height=20
 @seed = 1
      @octaves = 4
      @persistence = 0.25

 def raw_noise_2d(x, y)
      n = (x + y * 57).to_i
      n = (n << 13) ^ n
      (1.0 - ((n * (n * n * 15731 * @seed + 789221 * @seed) + 1376312589 * @seed) & 0x7fffffff) / 1073741824.0)
    end
    
   def smooth_noise_2d(x, y)
      corners = raw_noise_2d(x - 1, y - 1) + raw_noise_2d(x - 1, y + 1) + raw_noise_2d(x + 1, y - 1) + raw_noise_2d(x + 1, y + 1)
      sides = raw_noise_2d(x, y - 1) + raw_noise_2d(x, y + 1) + raw_noise_2d(x - 1, y) + raw_noise_2d(x + 1, y)
      center = raw_noise_2d(x, y)

      center / 4 + sides / 8 + corners / 16
    end

          def cosine_interpolate(a, b, x)
      f = (1 - Math.cos(x * Math::PI)) / 2
      a * (1 - f) + b * f
    end

    # alias interpolate linear_interpolate
    alias interpolate cosine_interpolate
    
   def interpolate_noise_2d(x, y)
      a = interpolate(smooth_noise_2d(x.floor, y.floor), smooth_noise_2d(x.floor + 1, y.floor), x - x.floor)
      b = interpolate(smooth_noise_2d(x.floor, y.floor + 1), smooth_noise_2d(x.floor + 1, y.floor + 1), x - x.floor)
      interpolate(a, b, y - y.floor)
    end
    
   def perlin_noise_2d(x, y)
      total = 0.0
      (0...@octaves).each do |i|
        frequency = 2.0 ** i
        amplitude = @persistence ** i
        total += interpolate_noise_2d(x * frequency, y * frequency) * amplitude
      end
      total
    end
    
   def perlin_noise_map(width, height)
      noise = Array.new(width)
      noise.map! { Array.new(height) }
      (0...width).each do |x|
        (0...height).each do |y|
          noise[x][y] = perlin_noise_2d(x, y)
          @dizi[x][y]=noise[x][y]
        end
      end
      noise
    end
 
   perlin_noise_map(@width, @height)
  
  for i in 0..@width-1
for j in 0..@height-1

pts1=[i,j,@dizi[i][j].to_f]
pts2=[i,j+1,@dizi[i][j+1].to_f]
pts3=[i+1,j+1,@dizi[i+1][j+1].to_f]
facee1=@ent.add_face pts1 ,pts2 ,pts3
pts11=[i,j,@dizi[i][j].to_f]
pts12=[i+1,j,@dizi[i+1][j].to_f]
pts13=[i+1,j+1,@dizi[i+1][j+1].to_f]
facee2=@ent.add_face pts11 ,pts12 ,pts13
facee1.back_material=[255,0,0]
facee2.back_material=[0,0,150]
end
end
 