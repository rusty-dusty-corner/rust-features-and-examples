
The purpose here is to show how the CPS style allows the use of pointers within 'local memory', while at the same time having a set of 'arrow' statements from which to declare an abstract program. Using 'Spec' to reduce the 'lifetime hell' which allows you to have a real working CPS in the abstract. This program is shown without 'arrow shugar', here you can see how it works in its natural form, how cursors are passed, how implementations for 'closure' are created directly in it. Sugar can be so powerful that it almost feels like a "rusty" code, but more on that later.

Part of the code that shows the behavior that should happen.
```rust
fn main() {
    use rand::Rng;
    let mut rng = rand::thread_rng();
    let x: u32 = rng.gen();
    println!("{}", program_asm(x >> 2));
}

#[inline(never)]
pub extern "C" fn program_asm(a: u32) -> u64 {
    let (_, b) = program(MyCursor);
    b.sfno((a,))
}

fn program<Cur>(
    cur: Cur,
) -> (
    Cur,
    ReturnSolOf<Cur, impl Attic<Clause = Cur::Clause, Domain = Own<u32>, Codomain = Own<u64>>>,
)
where
    Cur: IdOp + CatOp + ArrOp + AsRefOp + ReturnOp,
{
    decl_cfnom! { Cfn01 self f [] [Own<u32>] [Own<(u32,u64,u128)>] [
      SfnoWrap(|dom: u32| f.sfno((((dom + 11) * 22, ((dom + 33) as u64) * 44, ((dom + 55) as u128) * 66),)))
    ]}
    let (cur, cratic_a) = cur.arr_op(Cfn01);

    decl_cfnom! { Cfn02 self f [] [Ref<(u32,u64,u128)>] [Own<u64>] [
      SfnoWrap(|dom: &(u32,u64,u128)| f.sfno((dom.0 as u64 + dom.1 + dom.2 as u64 + 77_u64,)))
    ]}
    let (cur, cratic_b) = cur.arr_op(Cfn02);

    let (cur, cratic_c) = cur.as_ref_op(cratic_b);

    let (cur, cratic_d) = cur.id_op();

    let (cur, cratic_e) = cur.cat_op(cratic_a, cratic_c);

    let (cur, cratic_f) = cur.cat_op(cratic_e, cratic_d);

    let (cur, cratic) = cur.return_op(cratic_f);

    (cur, cratic)
}
```

* [You can see and run example on playground, also you can view asm](https://play.rust-lang.org/?version=nightly&mode=release&edition=2021&gist=eca8908d9bbc474f5a2682fa05db3f31)
* [Same code on gist](https://gist.github.com/rust-play/eca8908d9bbc474f5a2682fa05db3f31)

This is copy of asm output from playground.
```
playground::program_asm: # @playground::program_asm
	leal	(%rdi,%rdi,4), %eax
	leal	(%rdi,%rax,4), %eax
	addl	%edi, %eax
	addl	$242, %eax
	leal	33(%rdi), %ecx
	imulq	$44, %rcx, %rcx
	addl	$55, %edi
	movq	%rdi, %rdx
	shlq	$6, %rdx
	leaq	(%rdx,%rdi,2), %rdx
	addq	%rcx, %rax
	addq	%rdx, %rax
	addq	$77, %rax
	retq
```

The ability of the rust compiler in release mode to make such code reductions is truly amazing.
Implementations are made inside 'const _: ()' blocks, which makes the padding inaccessible, i.e. it is the implementation that works.
The code of these implementations seems to be quite extensive, the readability is almost at the limit, but this is not a problem, since in practise different 'proc-macro' and so on are used.

