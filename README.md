# Space Shooter Pico-8

```lua


function _init()
   t=0

   ship = {
      sp=1,
      x=60,
      y=100,
      h=4,
      p=0,
      t=0,
      imm=false,
      box = {x1=0,y1=0,x2=7,y2=7}
   }

   bullets = {}
   enemies = {}
   explosions = {}
   stars = {}

   for i=1,128 do
      add(stars,{
         x=rnd(128),
         y=rnd(128),
         s=rnd(10)+1
      })
   end

   start()
end

function respawn()
   local n = flr(rnd(9))+2
   for i=1,n do
      local d = -1
      if rnd(1)<0.5 then d=1 end
      add(enemies, {
         sp=17,
         m_x=i*16,
         m_y=-20-i*8,
         d=d,
         x=-32,
         y=-32,
         r=12,
         box = {x1=0,y1=0,x2=7,y2=7}
      })
   end

end



function start()
   _update = update_game
   _draw = draw_game
end

function game_over()
   _update = update_over
   _draw = draw_over
end

function update_over()

end

function draw_over()
   cls()
   print("game over",45,62,7)
end


function abs_box(s)
   local box = {}
   box.x1 = s.box.x1 + s.x
   box.y1 = s.box.y1 + s.y
   box.x2 = s.box.x2 + s.x
   box.y2 = s.box.y2 + s.y
   return box

end

function coll(a,b)
   -- todo
   local box_a = abs_box(a)
   local box_b = abs_box(b)

   if box_a.x1 > box_b.x2 or
   box_a.y1 > box_b.y2 or
   box_b.x1 > box_a.x2 or
   box_b.y1 > box_a.y2 then
      return false
   end


   return true


end

function explode(x,y)
   add(explosions,{x=x,y=y,t=0})
end

function fire()
   local b = {
      sp=3,
      x=ship.x,
      y=ship.y,
      dx=0,
      dy=-3,
      box = {x1=2,y1=0,x2=5,y2=4}
   }
   add(bullets,b)
end

function update_game()
   t=t+1
   if ship.imm then
      ship.t += 1
      if ship.t > 30 then
         ship.imm = false
         ship.t = 0
      end
   end


   for st in all(stars) do
      st.y += st.s
      if st.y >= 128 then
         st.y = 0
         st.x=rnd(128)
      end
   end

   for ex in all(explosions) do
      ex.t+=1
      if ex.t == 13 then
         del(explosions, ex)
      end
   end

   if #enemies <= 0 then
      respawn()
   end

   for e in all(enemies) do
      e.m_y += 8.3
      e.x = e.r*sin(e.d*t/60) + e.m_x
      e.y = e.r*cos(t/60) + e.m_y
      if coll(ship,e) and not ship.imm then
         ship.imm = true
         ship.h -= 1
         if ship.h <= 0 then
            game_over()
         end

      end

      if e.y > 150 then
         del(enemies,e)
      end

   end

   for b in all(bullets) do
      b.x+=b.dx
      b.y+=b.dy
      if b.x < 0 or b.x > 128 or
      b.y < 0 or b.y > 128 then
         del(bullets,b)
      end
      for e in all(enemies) do
         if coll(b,e) then
            del(enemies,e)
            ship.p += 1
            explode(e.x,e.y)
         end
      end
   end
   if(t%6<3) then
      ship.sp=1
   else
      ship.sp=2
   end

   if btn(0) then ship.x-=5.0 end
   if btn(1) then ship.x+=5.0 end
   if btn(2) then ship.y-=5.0 end
   if btn(3) then ship.y+=5.0 end
   if btnp(4) then fire() end
end


function draw_game()
   cls()
   for st in all(stars) do
      pset(st.x,st.y,5)
   end

   print(ship.p, 10, 5, 7)
   if not ship.imm or t%8 < 4 then
      spr(ship.sp,ship.x,ship.y)
   end

   for ex in all(explosions) do
      circ(ex.x,ex.y,ex.t/2,8+ex.t%3)
   end

   for b in all(bullets) do
      spr(b.sp,b.x,b.y)
   end

   for e in all(enemies) do
      spr(e.sp,e.x,e.y)
   end

   for i = 1, 4 do
      if i <= ship.h then
         spr(33, 90+6*i, 6)
      else
         spr(34, 90+6*i, 6)
      end
   end
end

```
