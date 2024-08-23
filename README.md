# Assessment
import 'package:flutter/material.dart';
import 'dart:math';

class CircularDashedProgressIndicator extends StatefulWidget {
  final double progress;
  final Color filledColor;
  final Color unfilledColor;
  final Duration duration;

  const CircularDashedProgressIndicator({
    required this.progress,
    required this.filledColor,
    required this.unfilledColor,
    this.duration = const Duration(milliseconds: 500),
  });

  @override
  _CircularDashedProgressIndicatorState createState() => _CircularDashedProgressIndicatorState();
}

class _CircularDashedProgressIndicatorState extends State<CircularDashedProgressIndicator> with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _animation;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(vsync: this, duration: widget.duration);
    _animation = Tween<double>(begin: 0, end: widget.progress).animate(_controller)
      ..addListener(() {
        setState(() {});
      });
    _controller.forward();
  }

  @override
  void didUpdateWidget(CircularDashedProgressIndicator oldWidget) {
    super.didUpdateWidget(oldWidget);
    if (oldWidget.progress != widget.progress) {
      _controller.duration = widget.duration;
      _animation = Tween<double>(begin: _animation.value, end: widget.progress).animate(_controller);
      _controller.forward(from: 0.0);
    }
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return CustomPaint(
      size: Size(100, 100),
      painter: _DashedCirclePainter(
        progress: _animation.value,
        filledColor: widget.filledColor,
        unfilledColor: widget.unfilledColor,
      ),
    );
  }
}

class _DashedCirclePainter extends CustomPainter {
  final double progress;
  final Color filledColor;
  final Color unfilledColor;

  _DashedCirclePainter({
    required this.progress,
    required this.filledColor,
    required this.unfilledColor,
  });

  @override
  void paint(Canvas canvas, Size size) {
    Paint paint = Paint()
      ..style = PaintingStyle.stroke
      ..strokeWidth = 8.0;

    double radius = min(size.width / 2, size.height / 2);
    Offset center = Offset(size.width / 2, size.height / 2);

    // Draw filled arc
    double sweepAngle = 2 * pi * progress;
    paint.color = filledColor;
    canvas.drawArc(Rect.fromCircle(center: center, radius: radius), -pi / 2, sweepAngle, false, paint);

    // Draw dashed unfilled arc
    paint.color = unfilledColor;
    Path unfilledPath = Path();

    const int dashCount = 40;
    const double dashWidth = 4.0;
    const double dashSpace = 2.0;

    for (int i = 0; i < dashCount; i++) {
      double startAngle = sweepAngle + (i * (dashWidth + dashSpace)) / radius;
      double endAngle = startAngle + dashWidth / radius;
      unfilledPath.addArc(Rect.fromCircle(center: center, radius: radius), startAngle, endAngle - startAngle);
    }

    canvas.drawPath(unfilledPath, paint);
  }

  @override
  bool shouldRepaint(covariant CustomPainter oldDelegate) {
    return true;
  }
}
