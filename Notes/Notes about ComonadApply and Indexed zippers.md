The 'I' functor is a sum monoid on the underlying Enum type's Int representation, as well as trivial Enum and Num based on the underlying type. This is a (somewhat) unprincipled way to make an Indexed zipper a law-obeying member of ComonadApply -- since we already need each element of the index to be an Enum to use duplicate, this doesn't add any extra constraint to the types of indices we'll end up using. Specifically, the problem this solves is that we need indices to behave like a monoid (i.e. we can't just pick one zipper's index) to obey the interchange law that u <@> pure y = pure ($ y) <@> u.

newtype I a = I a deriving ( Functor , Eq , Ord , Show )

type IZ  c       = Indexed (Identity (I c))  Z
type IZ2 c r     = Indexed (I c,I r)         Z2
type IZ3 c r l   = Indexed (I c,I r,I l)     Z3
type IZ4 c r l s = Indexed (I c,I r,I l,I s) Z4

instance (Monoid a) => Monoid (Identity a) where
   mempty = Identity mempty
   (Identity a) `mappend` (Identity b) =
      Identity (a `mappend` b)

instance (Enum a) => Enum (Identity a) where
   fromEnum (Identity a) = fromEnum a
   toEnum a = Identity (toEnum a)

instance (Enum a) => Enum (I a) where
   fromEnum (I a) = fromEnum a
   toEnum a = I (toEnum a)

instance (Enum a) => Monoid (I a) where
   mempty                = I $ toEnum 0
   (I a) `mappend` (I b) = I $ toEnum (fromEnum a + fromEnum b)

instance (Num a) => Num (I a) where
   (I a) + (I b) = I (a + b)
   (I a) * (I b) = I (a * b)
   abs    (I a)  = I (abs a)
   signum (I a)  = I (signum a)
   negate (I a)  = I (negate a)
   fromInteger   = I . fromInteger

 Update: I *really* don't like this approach, now that I think about it. Here's how it should work: for each coordinate in the indices, move the zipper with the *lesser* coordinate to the position of the zipper with the *greater* coordinate. We calculate "greater" and "lesser" based on the absolute value of the fromEnum of the index types. We must take the max of each coordinate to satisfy the identity law (pure id <*> v = v), as our definition of pure gives an index at the (fromEnum 0) of each index. Once the zippers are aligned, we can use the now-the-same index as our resultant index, and use the ComonadApply instance of the unindexed zippers to apply them to one another. This satisfies the Applicative laws.

 Update 2: ALL OF THESE PROBLEMS ARE BECAUSE WE ALLOWED PURE!!! If there is no pure (i.e. our Indexed zippers are ComonadApply but not Applicative) then the interchange law doesn't matter, and all we have to satisfy are the ComonadApply laws, which are perfectly happy given an implementation which simply grabs the index of its second argument. No pure means no interchange, means everything works perfectly. It would work equally well to choose the "function" index or the "value" index as the one to preserve; we choose arbitrarily to take the value index.
 
 Update 3: As noted in e0b8c2a11e25ed83730e0db0fc8e9e3936baf7ed, the previous "arbitrary" choice of keeping the "value" index causes nontermination when evaluating indexed things. This makes total sense. I fixed it.
